---
title: "[Internship] Customizing the OCR Engine in the Docling OCR Pipeline"
date: 2025-02-28 14:50:00 +09:00
last_modified_at: 2025-02-28 14:50:00 +09:00
categories: [Internship]
tags: [internship, langgraph, docling ocr, ocr, upstage]
description: "How to integrate a custom OCR model into the Docling OCR pipeline as a plugin."
language: en
postid: 12
---

## Problem Statement
- The default **Rapid OCR** provided by the Docling OCR pipeline shows relatively low recognition accuracy for Korean text.
- Since Docling is built as a modular pipeline, it supports the registration and utilization of custom OCR engines.
- Although this feature is briefly mentioned in the official documentation, the exact implementation details are not fully provided.

## Implementation Overview
Integrating an external OCR engine into Docling involves the following steps:

1. Package the desired OCR engine as a **pip-installable plugin**.
2. Write the registration code following the Docling standard interface.
3. Install the created plugin locally using the `pip install -e` command.
4. Enable the external plugin option in the Docling configuration and execute the pipeline.

---

## 1. Building the OCR Engine as a Plugin
The recommended folder structure is as follows:

```text
upstage_plugin_option/
├── upstage_plugin_option/
│   ├── upstage_plugin_option.py  # Main OCR logic
│   └── plugin.py                 # Entry point for Docling registration
└── pyproject.toml                # Package configuration file

```

#### 1) pyproject.toml

Configure the entry point so that Docling can recognize the package as a plugin.

```toml
[tool.poetry.plugins."docling"]
your_plugin_name = "upstage_plugin_option.plugin"

```

#### 2) plugin.py

This file must contain a specific function format that the Docling system calls to load the engine.

```python
def ocr_engines():
    from upstage_plugin_option.upstage_plugin_option import YourOcrModel
    return {
        "ocr_engines": [
            YourOcrModel,
        ]
    }

```

* If registration fails, it is recommended to check the [defaults.py](https://github.com/docling-project/docling/blob/main/docling/models/plugins/defaults.py) source code in the Docling repository to ensure the data structure matches.

#### 3) upstage_plugin_option.py

Implement the logic for the actual OCR call and result return. The following resources were used for implementation:

* [Docling Documentation - OCR Model Implementation](https://github.com/docling-project/docling/blob/main/docling/models/stages/ocr)

> **⚠️ Important:** When converting the OCR coordinates into the `TextCell` format, they must be **perfectly aligned with Docling’s layout detection coordinates**. If the coordinates are mismatched, table cell recognition and overall text layout will be completely broken.

---

## 2. Installation using pip install -e

Install the plugin in editable mode to ensure that changes are reflected immediately during development.

```bash
pip install -e ./upstage_plugin_option --no-build-isolation

```
* Enter the path where your `pyproject.toml` file is located for `./upstage_plugin_option`.
* Using the `--no-build-isolation` flag is recommended to prevent dependency conflicts within an existing virtual environment.


## Future Updates

Based on this guide, I plan to provide more in-depth content in the future, including specific integration code for the Upstage OCR API and coordinate correction logic.


-
