---
title: "[인턴] Docling OCR 파이프라인에서 OCR 엔진 커스텀 하기"
date: 2025-02-28 14:50:00 +09:00
last_modified_at: 2025-02-28 14:50:00 +09:00
categories: [Internship]
tags: [인턴, langgraph, internship, docling ocr, ocr, upstage]
description: "Docling OCR 파이프라인에 기본 엔진 대신 원하는 OCR 모델을 플러그인 형태로 통합하는 방법"
language: ko
postid: 12
---

## 문제 상황
- Docling OCR 파이프라인에서 기본 제공하는 **Rapid OCR**은 한글 인식률이 상대적으로 낮다.
- Docling은 모듈화된 파이프라인 구조를 갖고 있어 사용자 정의 OCR 등록을 지원하지만, 공식 문서에 구체적인 구현 방식이 상세히 나와 있지 않다.

## 이식 방법 요약
외부 OCR 엔진을 Docling에 이식하는 과정은 다음과 같다.

1. 사용할 OCR 엔진을 `pip install`이 가능한 **플러그인 형태**로 패키징한다.
2. Docling의 표준 규격에 맞추어 OCR 등록을 위한 코드를 작성한다.
3. `pip install -e` 명령어를 사용하여 제작한 플러그인을 로컬에 설치한다.
4. Docling 실행 시 외부 플러그인 사용 옵션을 활성화하고 모델을 변경하여 실행한다.

---

### 1. 플러그인 형태로 OCR 엔진 구축
전체적인 폴더 구조는 다음과 같이 구성한다.

```text
upstage_plugin_option/
├── upstage_plugin_option/
│   ├── upstage_plugin_option.py  # 메인 OCR 로직
│   └── plugin.py                 # Docling 등록용 진입점
└── pyproject.toml                # 패키지 설정 파일

```

#### 1) pyproject.toml

Docling이 해당 패키지를 플러그인으로 인식할 수 있도록 엔트리 포인트를 설정한다.

```toml
[tool.poetry.plugins."docling"]
your_plugin_name = "upstage_plugin_option.plugin"
```

#### 2) plugin.py

Docling 시스템이 엔진을 로드할 때 호출하는 함수 형식을 준수해야 한다.

```python
def ocr_engines():
    from upstage_plugin_option.upstage_plugin_option import YourOcrModel
    return {
        "ocr_engines": [
            YourOcrModel,
        ]
    }

```

* 등록이 정상적으로 되지 않는다면 Docling 내부의 [defaults.py](https://github.com/docling-project/docling/blob/main/docling/models/plugins/defaults.py) 소스코드를 참고하여 데이터 구조를 맞추는 것이 좋다.

#### 3) upstage_plugin_option.py

실제 OCR 호출 및 결과 반환 로직을 작성한다. 구현 시 다음 문서를 참고하였다.

* [Docling 공식 문서 - OCR 모델 구현체](https://github.com/docling-project/docling/blob/main/docling/models/stages/ocr)

> **⚠️ 주의사항:** OCR 모델의 좌표 결과를 `TextCell` 형식으로 변환할 때, **Docling의 레이아웃 디텍션 좌표계**와 정확히 일치시켜야 한다. 좌표가 어긋나면 표(Table)의 셀 인식과 텍스트 레이아웃이 완전히 깨질 수 있다.


## 2. pip install -e 를 사용하여 설치

개발 중인 플러그인을 즉시 반영하기 위해 에디터블 모드로 설치를 진행한다.

```bash
pip install -e ./upstage_plugin_option --no-build-isolation

```

* `/upstage_plugin_option` 위치에 자신의 `pyproject.toml` 파일이 있는 경로를 입력한다.
* 기존 가상환경 의존성과의 충돌을 방지하기 위해 `--no-build-isolation` 옵션을 사용하는 것이 안전하다.


## 향후 업데이트 예정

이 가이드를 바탕으로 구체적인 Upstage OCR API 연동 코드와 좌표 보정 로직을 포함한 심화 내용을 추후 업데이트할 예정이다.
