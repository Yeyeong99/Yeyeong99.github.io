---
title: "[인턴] Docling OCR 파이프라인에서 OCR 엔진 커스텀 하기"
date: 2025-02-28 14:50:00 +09:00
last_modified_at: 2025-02-28 14:50:00 +09:00
categories: [Internship]
tags: [인턴, LangGraph, Internship, Docling OCR, OCR, Upstage]
description: Docling OCR 파이프라인에서 기본으로 제공하는 OCR 외에 다른 OCR 모델을 붙여보자
language: ko
postid: 12
---

# 문제 상황
- Docling OCR 파이프라인에서 기본 제공하는 Rapid OCR의 경우 한글 인식률이 떨어짐
- Docling은 파이프라인에 해당하므로 사용자의 OCR 등록 및 활용을 지원함
- 해당 방법은 공식 문서에서 간단하게 소개되어 있지만 정확한 방식은 나와있지 않음


# 이식 방법
요약하면 다음과 같다.

1. 자신이 사용할 OCR 엔진을 pip install이 가능한 플러그인으로 만든다.
2. Docling OCR 파이프라인 공식문서에서 제공하는 방식대로 형식에 맞추어 등록을 위한 코드를 작성한다.
3. pip install -e 명령어를 이용해 제작한 플러그인을 설치한다.
4. Docling OCR 파이프라인 실행 시 외부 플러그인 사용 여부를 True 설정하고 형식에 맞추어 변경해 실행한다.


## 1. 플러그인 형태로 OCR 엔진 구축
폴더 구조를 다음과 같이 할 수 있다.

upstage_plugin_option
  ㄴ upstage_plugin_option
      ㄴ upstage_plugin_option.py
      ㄴ plugin.py
  ㄴ pyproject.toml

### 1) pyproject.toml

[tool.poetry.plugins."docling"]
your_plugin_name = "your_package.module"

- 플러그인으로 설치할 폴더 안에 pyproject.toml을 포함시키고 위와 같은 내용으로 구성한다.

### 2) plugin.py

def ocr_engines():
    from your_model_path import YourOcrModel
    return {
        "ocr_engines": [
            YourOcrModel,
        ]
    }

- Docling이 실행되며 ocr 엔진으로 인식하기 위해선 위와 같은 코드가 있어야 한다. 형식을 맞춰야 한다.
- 등록이 잘 되지 않는다면 형식 문제일 수 있으니 다음 공식문서 참고를 추천한다: [Docling 공식 문서 - default plugins](https://github.com/docling-project/docling/blob/main/docling/models/plugins/defaults.py)

### 3) upstage_plugin_option.py
다음 문서를 참고해서 코드를 작성하였다.
- [Docling 공식 문서 - OCR 모델](https://github.com/docling-project/docling/blob/main/docling/models/stages/ocr)
- ** 가장 주의해야 할 점 **: 자신이 쓰는 OCR 모델의 좌표 결과를 TextCell 형식으로 만들 때 Docling 의 레이아웃 디텍션 좌표와 맞아야 표의 셀과 레이아웃이 어긋나지 않음!

## 2. pip install -e 를 사용해 설치

pip install -e /upstage_plugin_option --no-build-isolation

- /upstage_plugin_option 에 자신의 pyproject.toml 파일이 있는 경로를 입력한다.
- 이미 가상환경이 구축되거나 하는 경우 충돌이 날 수 있다. 이런 경우 --no-build-isolation 을 쓰면 충돌을 없앨 수 있다.


더 자세한 내용은 추후 업데이트

