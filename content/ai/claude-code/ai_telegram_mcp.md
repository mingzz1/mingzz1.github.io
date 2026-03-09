---
title: "[Claude Code] Telegram MCP로 OpenClaw 느낌 내기"
date: 2026-03-05
draft: false
category: [ai]
subcategories: [claude-code]
tags: [ai, claude-code, telegram, mcp]
---

얼마 전 내가 OpenClaw를 사용한다고 글을 쓴 적이 있다. 메신저 하나로 AI가 알아서 모든 것을 다 해주는 경험은 정말 새로웠다. 그런데 토큰 정책 문제로 고민을 하다가 Telegram MCP를 새로 구축하게 되었다.

<!--more-->

OpenClaw 자체는 정말 혁신적이었다. 나를 위해 항상 대기하고 있는 비서 같은 느낌. 문제는 비용과 정책이었다.

첫 번째로, **Claude Code의 OAuth 토큰 사용은 정책 위반**이다. 이전 글에서도 언급했지만, Claude 구독자가 OAuth 토큰을 OpenClaw 같은 외부 에이전트에 연결해서 사용하는 것은 Anthropic의 TOS에 위배될 수 있다. 실제로 차단당했다는 사례도 최근 발생했다.

그렇다면 API를 쓰면 되지 않느냐 하면, 맞다. 이것이 정석적인 방법이다. 하지만 API 비용이 너무 많이 나온다. 대화가 쌓이면 쌓일수록 컨텍스트가 커지고, 그만큼 토큰이 기하급수적으로 소모된다. `/compact`로 정리하고 세션을 관리해도 일상적으로 쓰기에는 부담이 컸다. OpenClaw를 아끼고 아껴서 쓰다 보니 OpenClaw보다는 클로드 앱을 더 찾게 되고, 그러다 보니 굳이 OpenClaw를 서버 유지비 내가며 써야 하나?라는 생각이 들었다.

고민하던 중 떠오른 조합이 **Claude Code + MCP(Model Context Protocol)**였다. 이 조합의 핵심은 Claude Code가 직접 도구를 호출하는 구조라는 점이다. OAuth 토큰을 외부 서비스에 넘기는 것이 아니라 Claude Code가 MCP 서버를 통해 기능을 확장하는 방식이므로 정책 위반이 아니다.

인터넷에서 이미 구현되어 있는 Telegram MCP를 다운받아 쓸까 했는데, 어차피 Claude Code도 있겠다, 내가 직접 짜는 것도 아니니 그냥 Claude Code에게 시켜서 뚝딱 구축했다.

구조는 단순하다.

1. 텔레그램으로 메시지를 보내면 Webhook 서버가 수신
2. 서버에서 `claude --print`를 호출하여 응답 생성
3. 생성된 응답을 텔레그램으로 전송

크게 세 개의 파일로 구성된다.

| 파일 | 역할 |
|---|---|
| `telegram_mcp.py` | MCP 서버 - 메시지 조회/전송, 사용자 승인/취소 도구 제공 |
| `webhook_server.py` | Starlette 웹서버 - Webhook 수신, Claude 호출, 응답 전송 |
| `setup_webhook.py` | 텔레그램에 Webhook URL을 등록하는 일회성 스크립트 |

### 1. MCP 서버 (telegram_mcp.py)

MCP 서버는 Claude Code가 텔레그램과 상호작용할 수 있도록 했다. 오늘 받은 메시지 조회, 메시지 전송, 대화 기록 조회, 그리고 사용자 인가 관리까지 가능하다.

```python
#!/usr/bin/env python3

import asyncio
import json
import os
import random
import string
from datetime import datetime, timedelta, timezone
from typing import Any

import requests
from dotenv import load_dotenv
from mcp.server import Server
from mcp.server.stdio import stdio_server
import mcp.types as types

load_dotenv()

TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
AUTH_FILE = "/home/ubuntu/authorized_users.json"
ADMIN_CHAT_ID = os.getenv("ADMIN_CHAT_ID", "")


def get_api_url(method: str) -> str:
    return f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/{method}"


def telegram_request(method: str, params: dict = None) -> Any:
    """Telegram Bot API 공통 요청 함수"""
    if not TELEGRAM_BOT_TOKEN:
        raise ValueError("TELEGRAM_BOT_TOKEN이 .env 파일에 설정되지 않았습니다.")

    try:
        response = requests.post(
            get_api_url(method),
            json=params or {},
            timeout=30,
        )
        response.raise_for_status()
        data = response.json()

        if not data.get("ok"):
            raise ValueError(f"Telegram API 오류: {data.get('description', '알 수 없는 오류')}")

        return data.get("result", [])

    except requests.exceptions.ConnectionError:
        raise RuntimeError("네트워크 연결 실패. 인터넷 연결을 확인하세요.")
    except requests.exceptions.Timeout:
        raise RuntimeError("요청 시간 초과. 잠시 후 다시 시도하세요.")
    except requests.exceptions.HTTPError as e:
        raise RuntimeError(f"HTTP 오류: {e}")


# ── 인가 데이터 관리 ──────────────────────────────────────────

def load_auth() -> dict:
    """authorized_users.json 로드. 없으면 빈 구조 반환."""
    if os.path.exists(AUTH_FILE):
        with open(AUTH_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {"authorized": {}, "pending": {}}


def save_auth(auth: dict) -> None:
    """authorized_users.json 저장."""
    with open(AUTH_FILE, "w", encoding="utf-8") as f:
        json.dump(auth, f, ensure_ascii=False, indent=2)


def generate_code() -> str:
    """6자리 영숫자 연결 코드 생성."""
    chars = string.ascii_uppercase + string.digits
    return "".join(random.choices(chars, k=6))


def is_authorized(auth: dict, chat_id: str) -> bool:
    """chat_id가 승인된 사용자인지 확인."""
    return str(chat_id) in auth.get("authorized", {})


def is_pending(auth: dict, chat_id: str) -> bool:
    """chat_id가 이미 대기 중인지 확인."""
    for entry in auth.get("pending", {}).values():
        if str(entry.get("chat_id")) == str(chat_id):
            return True
    return False


def handle_unauthorized(auth: dict, chat_id: str, name: str) -> None:
    """미승인 사용자에게 연결 코드 발급 및 pending 등록."""
    if is_pending(auth, chat_id):
        return

    code = generate_code()
    while code in auth.get("pending", {}):
        code = generate_code()

    auth.setdefault("pending", {})[code] = {
        "chat_id": str(chat_id),
        "name": name,
        "created_at": datetime.now(timezone.utc).isoformat(),
    }
    save_auth(auth)

    try:
        telegram_request("sendMessage", {
            "chat_id": str(chat_id),
            "text": f"🔐 접근 권한이 필요합니다.\n\n연결 코드: {code}\n\n"
                    f"관리자에게 이 코드를 전달하여 승인을 요청하세요.",
        })
    except Exception:
        pass

    try:
        telegram_request("sendMessage", {
            "chat_id": ADMIN_CHAT_ID,
            "text": (
                f"🆕 새로운 사용자 접근 요청\n\n"
                f"이름: {name}\n"
                f"chat_id: {chat_id}\n"
                f"연결 코드: {code}\n\n"
                f"승인하려면 이 코드를 알려주세요."
            ),
        })
    except Exception:
        pass


def filter_authorized_updates(updates: list[dict]) -> list[dict]:
    """업데이트 목록에서 미승인 사용자를 처리하고 승인된 메시지만 반환."""
    auth = load_auth()
    authorized_updates = []

    for update in updates:
        message = (update.get("message") or update.get("channel_post")
                   or update.get("edited_message"))
        if not message:
            continue

        chat_id = str(message["chat"]["id"])

        if is_authorized(auth, chat_id):
            authorized_updates.append(update)
        else:
            sender = (message.get("from") or message.get("sender_chat") or {})
            name = (sender.get("first_name") or sender.get("username")
                    or "알 수 없음")
            handle_unauthorized(auth, chat_id, name)

    return authorized_updates


HISTORY_FILE = "/home/ubuntu/history.json"


def _load_history() -> dict:
    """history.json 로드. 없으면 빈 구조 반환."""
    if os.path.exists(HISTORY_FILE):
        with open(HISTORY_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {"conversations": [], "summary": None}


# ── 메시지 포맷팅 ─────────────────────────────────────────────

def format_message(update: dict) -> dict | None:
    """업데이트에서 메시지 정보 추출"""
    message = (update.get("message") or update.get("channel_post")
               or update.get("edited_message"))
    if not message:
        return None

    sender = message.get("from") or message.get("sender_chat") or {}
    return {
        "chat_id": message["chat"]["id"],
        "chat_name": (message["chat"].get("title")
                      or message["chat"].get("first_name", "")),
        "from": sender.get("username") or sender.get("first_name", "알 수 없음"),
        "text": message.get("text") or message.get("caption", "(텍스트 없음)"),
        "date": datetime.fromtimestamp(message["date"], tz=timezone.utc),
    }


# MCP 서버 초기화
server = Server("telegram-mcp")


@server.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="get_today_messages",
            description="오늘 받은 텔레그램 메시지 전체를 가져옵니다.",
            inputSchema={
                "type": "object",
                "properties": {},
                "required": [],
            },
        ),
        types.Tool(
            name="send_message",
            description="지정한 chat_id로 텔레그램 메시지를 보냅니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "chat_id": {
                        "type": "string",
                        "description": "메시지를 보낼 채팅 ID",
                    },
                    "text": {
                        "type": "string",
                        "description": "보낼 메시지 내용",
                    },
                },
                "required": ["chat_id", "text"],
            },
        ),
        types.Tool(
            name="get_chat_history",
            description="최근 N개의 텔레그램 메시지를 가져옵니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "limit": {
                        "type": "integer",
                        "description": "가져올 메시지 수 (1~100, 기본값: 10)",
                        "default": 10,
                        "minimum": 1,
                        "maximum": 100,
                    },
                },
                "required": [],
            },
        ),
        types.Tool(
            name="get_pending_users",
            description="승인 대기 중인 사용자 목록을 조회합니다.",
            inputSchema={
                "type": "object",
                "properties": {},
                "required": [],
            },
        ),
        types.Tool(
            name="approve_user",
            description="연결 코드로 사용자를 승인합니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "code": {
                        "type": "string",
                        "description": "6자리 연결 코드",
                    },
                },
                "required": ["code"],
            },
        ),
        types.Tool(
            name="revoke_user",
            description="승인된 사용자의 접근을 취소합니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "chat_id": {
                        "type": "string",
                        "description": "접근을 취소할 사용자의 chat_id",
                    },
                },
                "required": ["chat_id"],
            },
        ),
    ]


@server.call_tool()
async def call_tool(name: str, arguments: dict[str, Any]) -> list[types.TextContent]:
    try:
        if name == "get_today_messages":
            history = _load_history()
            KST = timezone(timedelta(hours=9))
            today = datetime.now(KST).date()

            today_messages = []
            for conv in history.get("conversations", []):
                ts = datetime.fromisoformat(conv["timestamp"])
                ts_kst = (ts.astimezone(KST) if ts.tzinfo
                          else ts.replace(tzinfo=timezone.utc).astimezone(KST))
                if ts_kst.date() == today:
                    today_messages.append(conv)

            if not today_messages:
                return [types.TextContent(type="text",
                                          text="오늘 받은 메시지가 없습니다.")]

            lines = [f"오늘 받은 메시지 ({len(today_messages)}건)\n" + "─" * 40]
            for conv in today_messages:
                ts = datetime.fromisoformat(conv["timestamp"])
                ts_kst = (ts.astimezone(KST) if ts.tzinfo
                          else ts.replace(tzinfo=timezone.utc).astimezone(KST))
                time_str = ts_kst.strftime("%H:%M:%S")
                lines.append(
                    f"[{time_str}] {conv['user']} | chat_id: {conv['chat_id']}\n"
                    f"  사용자: {conv['message']}\n"
                    f"  응답: {conv['response']}"
                )
            return [types.TextContent(type="text", text="\n".join(lines))]

        elif name == "send_message":
            chat_id = arguments.get("chat_id", "").strip()
            text = arguments.get("text", "").strip()

            if not chat_id:
                raise ValueError("chat_id가 필요합니다.")
            if not text:
                raise ValueError("text가 필요합니다.")

            telegram_request("sendMessage", {"chat_id": chat_id, "text": text})
            return [types.TextContent(
                type="text",
                text=f"메시지 전송 완료\nchat_id: {chat_id}\n내용: {text}",
            )]

        elif name == "get_chat_history":
            raw_limit = arguments.get("limit", 10)
            limit = max(1, min(int(raw_limit), 100))

            history = _load_history()
            conversations = history.get("conversations", [])
            messages = conversations[-limit:]

            if not messages:
                return [types.TextContent(type="text",
                                          text="가져올 메시지가 없습니다.")]

            lines = [f"최근 {len(messages)}개 메시지\n" + "─" * 40]
            for conv in messages:
                ts = datetime.fromisoformat(conv["timestamp"])
                time_str = ts.strftime("%Y-%m-%d %H:%M:%S")
                lines.append(
                    f"[{time_str}] {conv['user']} | chat_id: {conv['chat_id']}\n"
                    f"  사용자: {conv['message']}\n"
                    f"  응답: {conv['response']}"
                )
            return [types.TextContent(type="text", text="\n".join(lines))]

        elif name == "get_pending_users":
            auth = load_auth()
            pending = auth.get("pending", {})

            if not pending:
                return [types.TextContent(type="text",
                                          text="승인 대기 중인 사용자가 없습니다.")]

            lines = [f"승인 대기 ({len(pending)}명)\n" + "─" * 40]
            for code, info in pending.items():
                lines.append(
                    f"코드: {code} | 이름: {info['name']} "
                    f"| chat_id: {info['chat_id']}\n"
                    f"  요청일: {info['created_at']}"
                )
            return [types.TextContent(type="text", text="\n".join(lines))]

        elif name == "approve_user":
            code = arguments.get("code", "").strip().upper()
            if not code:
                raise ValueError("연결 코드가 필요합니다.")

            auth = load_auth()
            pending = auth.get("pending", {})

            if code not in pending:
                raise ValueError(f"유효하지 않은 연결 코드: {code}")

            user_info = pending.pop(code)
            chat_id = user_info["chat_id"]
            name = user_info["name"]

            auth.setdefault("authorized", {})[chat_id] = name
            save_auth(auth)

            try:
                telegram_request("sendMessage", {
                    "chat_id": chat_id,
                    "text": "✅ 접근이 승인되었습니다. 이제 봇을 사용할 수 있습니다.",
                })
            except Exception:
                pass

            return [types.TextContent(
                type="text",
                text=f"승인 완료\n이름: {name}\nchat_id: {chat_id}",
            )]

        elif name == "revoke_user":
            chat_id = arguments.get("chat_id", "").strip()
            if not chat_id:
                raise ValueError("chat_id가 필요합니다.")

            auth = load_auth()
            authorized = auth.get("authorized", {})

            if chat_id not in authorized:
                raise ValueError(f"승인된 사용자가 아닙니다: {chat_id}")

            name = authorized.pop(chat_id)
            save_auth(auth)

            return [types.TextContent(
                type="text",
                text=f"접근 취소 완료\n이름: {name}\nchat_id: {chat_id}",
            )]

        else:
            raise ValueError(f"알 수 없는 도구: {name}")

    except (ValueError, RuntimeError) as e:
        return [types.TextContent(type="text", text=f"오류: {e}")]
    except Exception as e:
        return [types.TextContent(type="text", text=f"예기치 않은 오류: {e}")]


async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            server.create_initialization_options(),
        )


if __name__ == "__main__":
    asyncio.run(main())
```

### 2. Webhook 서버 (webhook_server.py)

Webhook 서버는 텔레그램에서 메시지가 올 때마다 호출된다. 인가된 사용자 확인 후, `claude --print`를 호출하여 응답을 생성하고 다시 텔레그램으로 응답을 전송한다.

```python
#!/usr/bin/env python3

import asyncio
import json
import glob
import os
import re
import shutil
from datetime import datetime, timezone

import uvicorn
from dotenv import load_dotenv
from starlette.applications import Starlette
from starlette.requests import Request
from starlette.responses import JSONResponse
from starlette.routing import Route

from telegram_mcp import (
    load_auth, save_auth, is_authorized, is_pending,
    handle_unauthorized, telegram_request, ADMIN_CHAT_ID,
)

load_dotenv()

TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
WEBHOOK_URL = os.getenv("WEBHOOK_URL")
HISTORY_FILE = "/home/ubuntu/history.json"
WORKING_DIR = "/home/ubuntu"
SSL_CERT = "/home/ubuntu/webhook_cert.pem"
SSL_KEY = "/home/ubuntu/webhook_key.pem"


# ── History 관리 ──────────────────────────────────────────────

def load_history() -> dict:
    if os.path.exists(HISTORY_FILE):
        with open(HISTORY_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {"conversations": [], "summary": None}


def save_history(history: dict) -> None:
    with open(HISTORY_FILE, "w", encoding="utf-8") as f:
        json.dump(history, f, ensure_ascii=False, indent=2)


def estimate_tokens(text: str) -> int:
    """토큰 수 추정: 한글 1자 ≈ 2토큰, 영문 1단어 ≈ 1.3토큰"""
    korean = len(re.findall(r'[가-힣]', text))
    non_korean = re.sub(r'[가-힣]', '', text)
    words = len(non_korean.split())
    return int(korean * 2 + words * 1.3)


# ── 맥락 구성 ─────────────────────────────────────────────────

def build_context(history: dict) -> str:
    """이전 대화 맥락을 문자열로 구성"""
    parts = []

    if history.get("summary"):
        parts.append(f"[이전 대화 요약]\n{history['summary']}")

    convos = history.get("conversations", [])[-20:]
    if convos:
        parts.append("[최근 대화]")
        for c in convos:
            parts.append(f"사용자({c.get('user', '?')}): {c['message']}")
            parts.append(f"AI_AGENT_NAME: {c['response']}")

    return "\n".join(parts)


# ── claude --print 호출 ───────────────────────────────────────

async def call_claude(prompt: str, timeout: int = 120) -> str:
    env = os.environ.copy()
    env.pop("CLAUDECODE", None)

    proc = await asyncio.create_subprocess_exec(
        "claude", "--print", "--dangerously-skip-permissions", "-p", prompt,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
        cwd=WORKING_DIR,
        env=env,
    )

    try:
        stdout, stderr = await asyncio.wait_for(proc.communicate(), timeout=timeout)
    except asyncio.TimeoutError:
        proc.kill()
        await proc.wait()
        return "(오류: claude 응답 시간 초과)"

    if proc.returncode != 0:
        err = stderr.decode().strip()
        print(f"[ERROR] claude 실행 실패: {err}", flush=True)
        return f"(오류 발생: {err})"

    return stdout.decode().strip()


# ── 텔레그램 메시지 전송 ──────────────────────────────────────

def send_telegram(chat_id: str, text: str) -> None:
    """긴 메시지는 4096자 단위로 분할 전송"""
    max_len = 4096
    for i in range(0, len(text), max_len):
        telegram_request("sendMessage", {
            "chat_id": chat_id,
            "text": text[i:i + max_len],
        })


# ── 명령어 처리 ───────────────────────────────────────────────

async def handle_command(chat_id: str, text: str) -> str:
    """슬래시 명령어 처리. 응답 문자열 반환."""
    cmd = text.strip().lower()

    if cmd == "/clear":
        if os.path.exists(HISTORY_FILE):
            ts = datetime.now().strftime("%Y%m%d_%H%M")
            backup = f"/home/ubuntu/history_backup_{ts}.json"
            shutil.copy2(HISTORY_FILE, backup)
            save_history({"conversations": [], "summary": None})
            return f"대화 기록 초기화 완료.\n백업: {os.path.basename(backup)}"
        return "초기화할 대화 기록이 없습니다."

    if cmd == "/compact":
        history = load_history()
        convos = history.get("conversations", [])
        if not convos and not history.get("summary"):
            return "요약할 대화가 없습니다."

        context = build_context(history)
        summary = await call_claude(
            f"다음 대화 내용을 핵심만 간결하게 요약해줘. "
            f"중요한 맥락과 결정사항 위주로:\n\n{context}"
        )
        save_history({"conversations": [], "summary": summary})
        return f"대화 요약 완료.\n\n{summary}"

    if cmd == "/session":
        history = load_history()
        convos = history.get("conversations", [])
        total = len(convos)
        tokens = sum(c.get("tokens_estimated", 0) for c in convos)
        size = (os.path.getsize(HISTORY_FILE)
                if os.path.exists(HISTORY_FILE) else 0)
        last = convos[-1]["timestamp"] if convos else "없음"
        has_summary = "있음" if history.get("summary") else "없음"
        return (
            f"📊 세션 정보\n"
            f"대화 수: {total}건\n"
            f"누적 토큰(추정): {tokens}\n"
            f"파일 크기: {size:,} bytes\n"
            f"마지막 대화: {last}\n"
            f"요약: {has_summary}"
        )

    if cmd == "/backup list":
        files = sorted(glob.glob("/home/ubuntu/history_backup_*.json"))
        if not files:
            return "백업 파일이 없습니다."
        lines = ["📁 백업 목록"]
        for f in files:
            name = os.path.basename(f)
            size = os.path.getsize(f)
            lines.append(f"  {name} ({size:,} bytes)")
        return "\n".join(lines)

    if cmd.startswith("/backup restore"):
        parts = text.strip().split(maxsplit=2)
        if len(parts) < 3:
            return "사용법: /backup restore [파일명]"
        filename = parts[2]
        path = f"/home/ubuntu/{filename}"
        if not os.path.exists(path):
            return f"파일을 찾을 수 없습니다: {filename}"
        shutil.copy2(path, HISTORY_FILE)
        return f"복원 완료: {filename}"

    return f"알 수 없는 명령어: {text.strip()}"


# ── 메시지 처리 메인 ──────────────────────────────────────────

async def process_message(chat_id: str, user_name: str, text: str) -> None:
    """일반 메시지 처리: 맥락 구성 → claude 호출 → 저장 → 응답"""
    history = load_history()
    context = build_context(history)

    system = (
        "너는 AI_AGENT_NAME, mingzzi의 개인 비서야. "
        "지금은 텔레그램 채팅 모드야. 너의 출력이 그대로 사용자에게 "
        "텔레그램 메시지로 전송돼. "
        "메시지 전송은 시스템이 자동으로 처리하므로 네가 별도로 보낼 필요 없어. "
        "사용자에게 보낼 답장 내용만 출력해. "
        "한국어로 핵심만 간결하게 답변해."
    )

    if context:
        prompt = f"[시스템] {system}\n\n{context}\n\n사용자({user_name}): {text}"
    else:
        prompt = f"[시스템] {system}\n\n사용자({user_name}): {text}"

    response = await call_claude(prompt)

    entry = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "chat_id": str(chat_id),
        "user": user_name,
        "message": text,
        "response": response,
        "tokens_estimated": estimate_tokens(text) + estimate_tokens(response),
    }
    history["conversations"].append(entry)
    save_history(history)

    send_telegram(str(chat_id), response)


# ── Webhook 엔드포인트 ────────────────────────────────────────

async def webhook(request: Request) -> JSONResponse:
    try:
        data = await request.json()
    except Exception:
        return JSONResponse({"ok": False}, status_code=400)

    message = data.get("message")
    if not message:
        return JSONResponse({"ok": True})

    chat_id = str(message["chat"]["id"])
    sender = message.get("from", {})
    user_name = (sender.get("first_name") or sender.get("username")
                 or "알 수 없음")
    text = message.get("text", "").strip()

    if not text:
        return JSONResponse({"ok": True})

    auth = load_auth()
    if not is_authorized(auth, chat_id):
        handle_unauthorized(auth, chat_id, user_name)
        return JSONResponse({"ok": True})

    if text.startswith("/"):
        asyncio.create_task(_handle_command_task(chat_id, text))
    else:
        asyncio.create_task(_handle_message_task(chat_id, user_name, text))

    return JSONResponse({"ok": True})


async def _handle_command_task(chat_id: str, text: str) -> None:
    stop_typing = asyncio.Event()
    typing_task = asyncio.create_task(typing_indicator(chat_id, stop_typing))
    try:
        result = await handle_command(chat_id, text)
        send_telegram(chat_id, result)
    except Exception as e:
        send_telegram(chat_id, f"명령어 처리 오류: {e}")
    finally:
        stop_typing.set()
        await typing_task


async def health(request: Request) -> JSONResponse:
    return JSONResponse({"status": "ok", "service": "AI_AGENT_NAME"})


# ── Webhook 등록 ──────────────────────────────────────────────

async def register_webhook() -> None:
    if not WEBHOOK_URL:
        print("[WARNING] WEBHOOK_URL이 설정되지 않았습니다.")
        return

    url = WEBHOOK_URL.rstrip("/")
    if not url.endswith("/webhook"):
        url += "/webhook"

    try:
        import requests as _requests
        api_url = (f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}"
                   f"/setWebhook")

        if os.path.exists(SSL_CERT):
            with open(SSL_CERT, "rb") as cert_file:
                result = _requests.post(
                    api_url,
                    data={"url": url},
                    files={"certificate": cert_file},
                    timeout=30,
                ).json()
            print(f"[INFO] Webhook 등록 완료 (인증서 포함): {url} → {result}")
        else:
            result = telegram_request("setWebhook", {"url": url})
            print(f"[INFO] Webhook 등록 완료 (인증서 없음): {url} → {result}")
    except Exception as e:
        print(f"[ERROR] Webhook 등록 실패: {e}")


# ── 앱 설정 ───────────────────────────────────────────────────

app = Starlette(
    routes=[
        Route("/webhook", webhook, methods=["POST"]),
        Route("/health", health, methods=["GET"]),
    ],
    on_startup=[register_webhook],
)

if __name__ == "__main__":
    ssl_kwargs = {}
    if os.path.exists(SSL_CERT) and os.path.exists(SSL_KEY):
        ssl_kwargs["ssl_certfile"] = SSL_CERT
        ssl_kwargs["ssl_keyfile"] = SSL_KEY
        print(f"[INFO] SSL 활성화: cert={SSL_CERT}, key={SSL_KEY}")
    else:
        print("[WARNING] SSL 인증서/키 파일이 없어 HTTP로 실행합니다.")
    uvicorn.run(app, host="0.0.0.0", port=8443, **ssl_kwargs)
```

이렇게 구현하면 텔레그램을 통해 Claude Code를 사용할 수 있다. 한 가지 아쉬웠던 점은 OpenClaw를 쓸 때 메신저를 보내면 곧바로 `Typing...`이라고 뜨는 것이었다. 이를 통해 서버가 제대로 동작하는지, 답변을 생성 중인지를 확인할 수 있었는데, 위와 같이 구현하면 그런 게 보이지 않아 구별하기가 어려웠다. 그래서 `webhook_server.py`에 `typing_indicator`와 `_handle_message_task`를 추가하여 동일한 기능을 구현했다.

```python
async def typing_indicator(chat_id: str, stop_event: asyncio.Event) -> None:
    """답장 전송까지 4초마다 typing 상태를 갱신"""
    while not stop_event.is_set():
        try:
            telegram_request("sendChatAction", {
                "chat_id": chat_id,
                "action": "typing",
            })
        except Exception:
            pass
        try:
            await asyncio.wait_for(stop_event.wait(), timeout=4)
        except asyncio.TimeoutError:
            pass


async def _handle_message_task(chat_id: str, user_name: str, text: str) -> None:
    stop_typing = asyncio.Event()
    typing_task = asyncio.create_task(typing_indicator(chat_id, stop_typing))
    try:
        await process_message(chat_id, user_name, text)
    except Exception as e:
        print(f"[ERROR] 메시지 처리 실패: {e}", flush=True)
        send_telegram(chat_id, f"메시지 처리 오류: {e}")
    finally:
        stop_typing.set()
        await typing_task
```

텔레그램의 `sendChatAction` API는 typing 상태가 5초 후 자동으로 사라진다. 그래서 4초마다 갱신하도록 했다. `asyncio.Event`를 사용해서 응답이 완료되면 즉시 Typing을 멈추는 구조다. 이렇게 하면 나에게는 답변이 오기 전까지 계속 Typing이라는 표시가 뜬다.

<img src="/images/ai/ai_telegram_mcp/01.png" width="80%" />

최근 Claude의 업데이트가 많아지고 있다. **Claude Code Remote**가 출시되었고, 데스크톱 앱에는 cronjob 같은 **Cowork**, 크롬이나 MS Office 등 다른 도구와의 연동 기능도 추가되고 있다. 특히 Claude Code Remote는 원격 서버에서 Claude Code를 실행하고 로컬에서 접속하는 방식인데 현재는 Max 요금제에서만 가능해 테스트해 보진 못했다. Pro 요금제까지 확장 계획이라고 하던데, 그러면 굳이 Telegram 없이도 사용할 수 있을 것 같다.

<img src="/images/ai/ai_telegram_mcp/02.png" width="80%" />

<img src="/images/ai/ai_telegram_mcp/03.png" width="80%" />

나는 현재 Claude Code를 원격 서버에 띄워놓고 사용하고 있다. 그런데 데스크톱 앱을 보니 기능이 상당히 많이 추가된 것 같아 Mac Mini를 사서 본격적으로 모든 걸 연동해 두고 쓰고 싶어진다. 활용할 수 있는 방안이 아주 무궁무진할 것 같다.

괜히 저렴하게 살 수 있는 곳이 어디 없나 살펴보게 되는 요즘이다...
