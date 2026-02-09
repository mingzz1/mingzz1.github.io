---
title: "[Supply Chain] SBOM과 AIBOM: 소프트웨어 및 AI 공급망 보안의 핵심"
date: 2026-02-09
draft: false
category: [security]
subcategories: [supply-chain]
tags: [Security, SBOM, AIBOM, Supply Chain, Software Supply Chain]
---

소프트웨어 공급망 공격이 급증하면서, 우리가 사용하는 소프트웨어가 어떤 구성요소로 이루어져 있는지 파악하는 것이 그 어느 때보다 중요해졌다. 이 글에서는 소프트웨어 구성요소 목록인 SBOM과, AI 시대에 새롭게 등장한 AIBOM에 대해 알아본다.

<!--more-->

## 1. SBOM (Software Bill of Materials)

### 1.1 SBOM이란?

SBOM은 **Software Bill of Materials**의 약자로, 소프트웨어를 구성하는 모든 컴포넌트, 라이브러리, 모듈의 목록을 체계적으로 정리한 문서이다. 제조업에서 제품을 만들 때 사용하는 부품 목록(BOM)의 개념을 소프트웨어에 적용한 것이다.

쉽게 말해, 하나의 소프트웨어가 어떤 오픈소스 라이브러리를 사용하고, 각 라이브러리의 버전은 무엇이며, 라이선스는 어떤 것인지를 한눈에 볼 수 있도록 정리한 **소프트웨어의 성분표**라고 할 수 있다.

### 1.2 SBOM이 왜 필요한가?

SBOM이 주목받게 된 배경에는 여러 사건과 변화가 있다.

**1) 소프트웨어 공급망 공격의 증가**

- **SolarWinds 사건 (2020)**: SolarWinds의 Orion 소프트웨어 업데이트에 악성코드가 삽입되어 미국 정부기관을 포함한 수천 개 조직이 피해를 입었다. 이 사건을 계기로 미국 정부는 소프트웨어 공급망 보안을 강화하기 시작했다.
- **Log4Shell (2021)**: Apache Log4j의 치명적 취약점(CVE-2021-44228)이 발견되었을 때, 많은 기업이 자사 소프트웨어에 Log4j가 포함되어 있는지조차 파악하지 못해 대응이 지연되었다.
- **XZ Utils 백도어 (2024)**: 리눅스의 핵심 압축 유틸리티인 XZ Utils에 의도적인 백도어가 삽입된 사건으로, 오픈소스 공급망의 취약성을 다시 한번 보여주었다.

**2) 규제 및 정책의 강화**

- 2021년 미국 바이든 대통령의 **행정명령 14028 (Executive Order on Improving the Nation's Cybersecurity)**에서 연방 정부에 소프트웨어를 납품하는 업체에 SBOM 제출을 요구했다.
- EU의 **Cyber Resilience Act (CRA)**에서도 SBOM 제공을 의무화하는 방향으로 진행 중이다.

**3) 오픈소스 사용의 보편화**

현대 소프트웨어의 70~90%가 오픈소스 컴포넌트로 구성되어 있다. 이러한 상황에서 사용 중인 오픈소스의 취약점이나 라이선스 문제를 추적하려면 체계적인 목록 관리가 필수적이다.

### 1.3 SBOM 표준 포맷

SBOM에는 대표적으로 두 가지 표준 포맷이 있다.

**1) SPDX (Software Package Data Exchange)**

Linux Foundation에서 관리하는 국제 표준(ISO/IEC 5962:2021)이다.

```json
{
  "spdxVersion": "SPDX-2.3",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "Example-WebApp",
  "documentNamespace": "https://example.com/sbom/webapp-1.0",
  "creationInfo": {
    "created": "2026-02-09T00:00:00Z",
    "creators": ["Tool: example-sbom-generator-1.0"]
  },
  "packages": [
    {
      "SPDXID": "SPDXRef-Package-react",
      "name": "react",
      "versionInfo": "18.2.0",
      "supplier": "Organization: Meta Platforms",
      "downloadLocation": "https://registry.npmjs.org/react/-/react-18.2.0.tgz",
      "licenseConcluded": "MIT",
      "externalRefs": [
        {
          "referenceCategory": "PACKAGE-MANAGER",
          "referenceType": "npm",
          "referenceLocator": "react@18.2.0"
        }
      ]
    },
    {
      "SPDXID": "SPDXRef-Package-express",
      "name": "express",
      "versionInfo": "4.18.2",
      "supplier": "Organization: OpenJS Foundation",
      "downloadLocation": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "licenseConcluded": "MIT",
      "externalRefs": [
        {
          "referenceCategory": "SECURITY",
          "referenceType": "cpe23Type",
          "referenceLocator": "cpe:2.3:a:expressjs:express:4.18.2:*:*:*:*:*:*:*"
        }
      ]
    }
  ],
  "relationships": [
    {
      "spdxElementId": "SPDXRef-DOCUMENT",
      "relationshipType": "DESCRIBES",
      "relatedSpdxElement": "SPDXRef-Package-react"
    }
  ]
}
```

**2) CycloneDX**

OWASP에서 관리하는 포맷으로, 보안 관점에 특화되어 있다.

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "serialNumber": "urn:uuid:3e671687-395b-41f5-a30f-a58921a69b79",
  "version": 1,
  "metadata": {
    "timestamp": "2026-02-09T00:00:00Z",
    "component": {
      "type": "application",
      "name": "Example-WebApp",
      "version": "1.0.0"
    }
  },
  "components": [
    {
      "type": "library",
      "name": "react",
      "version": "18.2.0",
      "purl": "pkg:npm/react@18.2.0",
      "licenses": [
        {
          "license": {
            "id": "MIT"
          }
        }
      ],
      "hashes": [
        {
          "alg": "SHA-256",
          "content": "a1b2c3d4e5f6..."
        }
      ]
    },
    {
      "type": "library",
      "name": "express",
      "version": "4.18.2",
      "purl": "pkg:npm/express@4.18.2",
      "licenses": [
        {
          "license": {
            "id": "MIT"
          }
        }
      ]
    }
  ],
  "vulnerabilities": [
    {
      "id": "CVE-2024-XXXXX",
      "source": {
        "name": "NVD"
      },
      "ratings": [
        {
          "severity": "high",
          "score": 8.1,
          "method": "CVSSv3"
        }
      ],
      "affects": [
        {
          "ref": "pkg:npm/express@4.18.2"
        }
      ]
    }
  ]
}
```

### 1.4 SBOM 활용 방법

1. **취약점 관리**: SBOM에 기록된 컴포넌트를 NVD(National Vulnerability Database) 등과 대조하여 알려진 취약점을 빠르게 탐지하고 대응할 수 있다.
2. **라이선스 컴플라이언스**: 사용 중인 오픈소스의 라이선스 조건을 파악하고, 라이선스 위반 위험을 사전에 방지할 수 있다.
3. **인시던트 대응**: Log4Shell과 같은 취약점이 공개되었을 때, SBOM을 통해 영향받는 시스템을 즉시 식별할 수 있다.
4. **규제 준수**: 미국 연방 정부 조달이나 EU CRA 등 SBOM 제출이 요구되는 규제를 충족할 수 있다.
5. **공급망 투명성**: 소프트웨어 공급업체와 구매자 간의 투명성을 높여 신뢰 관계를 구축할 수 있다.

---

## 2. AIBOM (AI Bill of Materials)

### 2.1 AIBOM이란?

AIBOM은 **AI Bill of Materials**의 약자로, AI 시스템을 구성하는 모든 요소를 체계적으로 문서화한 목록이다. SBOM이 소프트웨어의 구성요소를 다룬다면, AIBOM은 여기에 더해 **AI 모델, 학습 데이터, 학습 파이프라인, 하이퍼파라미터** 등 AI 특유의 구성요소까지 포함한다.

AI 시스템은 전통적인 소프트웨어와 달리 코드만으로 동작이 결정되지 않는다. 어떤 데이터로 학습했는지, 어떤 모델 아키텍처를 사용했는지, 어떤 전처리 과정을 거쳤는지에 따라 동작이 크게 달라진다. AIBOM은 이러한 AI의 고유한 특성을 반영하여 투명성과 추적 가능성을 보장하기 위한 문서이다.

### 2.2 AIBOM이 왜 필요한가?

**1) AI 공급망의 복잡성**

현대 AI 시스템은 다양한 외부 요소에 의존한다.

- Hugging Face 등에서 다운로드한 **사전 학습 모델(Pre-trained Model)**
- 인터넷에서 수집하거나 데이터 제공업체로부터 구매한 **학습 데이터셋**
- PyTorch, TensorFlow 등의 **ML 프레임워크**
- 다양한 **전처리/후처리 파이프라인**

이러한 구성요소 중 하나라도 문제가 있으면 AI 시스템 전체가 영향을 받을 수 있다.

**2) AI 특유의 보안 위협**

- **데이터 포이즈닝(Data Poisoning)**: 학습 데이터에 악의적인 데이터를 삽입하여 모델의 동작을 조작하는 공격
- **모델 백도어(Model Backdoor)**: 특정 트리거 입력에 대해 의도된 오작동을 하도록 모델을 조작하는 공격
- **모델 탈취(Model Stealing)**: API를 통해 모델의 동작을 복제하는 공격

이런 위협에 대응하려면 AI 시스템의 각 구성요소를 추적할 수 있어야 한다.

**3) AI 규제의 등장**

- EU **AI Act**: AI 시스템을 위험도에 따라 분류하고, 고위험 AI에 대해 투명성 요구사항을 명시했다.
- 미국 **AI Executive Order (2023)**: AI 시스템의 안전성과 투명성을 강화하는 방향을 제시했다.
- 각국의 AI 규제가 강화되면서 AI 시스템의 구성요소를 체계적으로 문서화할 필요성이 높아지고 있다.

### 2.3 AIBOM 예시 형태

AIBOM은 아직 SBOM처럼 표준화된 단일 포맷이 확립되지는 않았지만, CycloneDX 1.5+에서 ML 컴포넌트를 지원하기 시작했으며, SPDX AI Profile도 개발 중이다. 아래는 CycloneDX 기반의 AIBOM 예시이다.

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "serialNumber": "urn:uuid:9a8b7c6d-5e4f-3a2b-1c0d-e9f8a7b6c5d4",
  "version": 1,
  "metadata": {
    "timestamp": "2026-02-09T00:00:00Z",
    "component": {
      "type": "machine-learning-model",
      "name": "Sentiment-Analysis-Service",
      "version": "2.1.0",
      "description": "고객 리뷰 감성 분석 AI 서비스"
    }
  },
  "components": [
    {
      "type": "machine-learning-model",
      "name": "sentiment-bert-ko",
      "version": "1.0.0",
      "description": "한국어 감성 분석을 위해 Fine-tuning된 BERT 모델",
      "purl": "pkg:huggingface/org/sentiment-bert-ko@1.0.0",
      "modelCard": {
        "modelParameters": {
          "approach": {
            "type": "supervised"
          },
          "task": "sentiment-analysis",
          "architectureFamily": "BERT",
          "modelArchitecture": "bert-base-multilingual-cased",
          "datasets": [
            {
              "name": "korean-review-dataset",
              "type": "dataset",
              "contents": {
                "type": "text",
                "sizeInBytes": 524288000
              },
              "classification": "public"
            }
          ],
          "inputs": [
            {
              "format": "text/plain"
            }
          ],
          "outputs": [
            {
              "format": "application/json"
            }
          ]
        },
        "quantitativeAnalysis": {
          "performanceMetrics": [
            {
              "type": "accuracy",
              "value": "0.92",
              "confidenceInterval": {
                "lowerBound": "0.90",
                "upperBound": "0.94"
              }
            },
            {
              "type": "f1-score",
              "value": "0.91"
            }
          ]
        },
        "considerations": {
          "ethicalConsiderations": [
            {
              "name": "bias",
              "mitigationStrategy": "학습 데이터에서 성별, 나이, 지역 관련 편향을 제거하기 위한 전처리 수행"
            }
          ]
        }
      }
    },
    {
      "type": "framework",
      "name": "pytorch",
      "version": "2.1.0",
      "purl": "pkg:pypi/torch@2.1.0"
    },
    {
      "type": "library",
      "name": "transformers",
      "version": "4.36.0",
      "purl": "pkg:pypi/transformers@4.36.0"
    },
    {
      "type": "data",
      "name": "korean-review-dataset",
      "version": "3.0",
      "description": "한국어 상품 리뷰 감성 분석 데이터셋 (50만 건)",
      "data": {
        "type": "dataset",
        "contents": {
          "url": "https://example.com/datasets/korean-review-v3"
        },
        "governance": {
          "owners": ["Data Team"],
          "custodians": ["ML Platform Team"]
        },
        "classification": "internal"
      }
    }
  ],
  "dependencies": [
    {
      "ref": "Sentiment-Analysis-Service",
      "dependsOn": [
        "sentiment-bert-ko",
        "pytorch",
        "transformers",
        "korean-review-dataset"
      ]
    }
  ]
}
```

위 예시에서 볼 수 있듯이, AIBOM은 기존 SBOM의 소프트웨어 라이브러리 정보에 더해 **모델 아키텍처, 학습 데이터셋, 성능 지표, 윤리적 고려사항** 등 AI 고유의 정보를 포함한다.

### 2.4 AIBOM 활용 방법

1. **AI 공급망 보안**: 사용 중인 사전 학습 모델과 데이터셋의 출처를 추적하여, 데이터 포이즈닝이나 모델 백도어 등의 위협을 탐지할 수 있다.
2. **모델 거버넌스**: AI 모델의 버전, 학습 이력, 성능 변화를 체계적으로 관리하여 모델 품질을 유지할 수 있다.
3. **규제 준수**: EU AI Act 등 AI 관련 규제에서 요구하는 투명성 및 문서화 요구사항을 충족할 수 있다.
4. **편향(Bias) 관리**: 학습 데이터의 구성과 출처를 문서화하여, AI 모델의 편향 문제를 식별하고 개선할 수 있다.
5. **재현 가능성(Reproducibility)**: 모델 학습에 사용된 데이터, 하이퍼파라미터, 프레임워크 버전 등을 기록하여, 실험과 결과를 재현할 수 있다.
6. **인시던트 대응**: AI 시스템에서 문제가 발생했을 때, AIBOM을 통해 어떤 모델이나 데이터가 원인인지 빠르게 파악할 수 있다.

---

## 3. SBOM vs AIBOM 비교

| 구분 | SBOM | AIBOM |
|------|------|-------|
| **대상** | 소프트웨어 컴포넌트 | AI 시스템 전체 (모델 + 데이터 + 소프트웨어) |
| **주요 구성요소** | 라이브러리, 패키지, 프레임워크 | 모델, 학습 데이터, ML 파이프라인, 하이퍼파라미터 |
| **표준** | SPDX, CycloneDX (성숙) | CycloneDX ML 확장, SPDX AI Profile (발전 중) |
| **주요 위협** | 취약한 라이브러리, 라이선스 위반 | 데이터 포이즈닝, 모델 백도어, 편향 |
| **규제** | 미국 EO 14028, EU CRA | EU AI Act, 미국 AI EO |
| **도구** | Syft, Trivy, OWASP Dependency-Track | 아직 초기 단계 |

---

## 4. 마무리

SBOM은 이미 소프트웨어 공급망 보안의 핵심 요소로 자리잡았고, AIBOM은 AI 시대에 맞춰 빠르게 발전하고 있다. 특히 AI 시스템이 의료, 금융, 자율주행 등 고위험 분야에 점점 더 많이 활용되면서, AI 공급망의 투명성과 추적 가능성을 보장하는 AIBOM의 중요성은 더욱 커질 것이다.

조직에서는 먼저 SBOM 체계를 구축하고, 이를 기반으로 AIBOM까지 확장하는 단계적 접근을 추천한다. 이미 Syft, Trivy, OWASP Dependency-Track 등 SBOM 생성 및 관리 도구가 성숙해 있으므로, 아직 SBOM을 도입하지 않은 조직이라면 지금이 시작하기 좋은 시점이다.

---

## References

- NTIA SBOM 가이드: https://www.ntia.gov/page/software-bill-of-materials
- SPDX 공식 사이트: https://spdx.dev/
- CycloneDX 공식 사이트: https://cyclonedx.org/
- EU AI Act: https://artificialintelligenceact.eu/
- OWASP Machine Learning Security Top 10: https://owasp.org/www-project-machine-learning-security-top-10/
