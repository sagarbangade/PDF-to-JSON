# PDF-to-JSON Converter

This repository provides Python scripts for converting PDF documents into structured JSON files. It uses libraries like `pdfplumber` and `pdfminer` for text extraction and formatting.

## Features

- Extract text and titles from PDFs based on font size and formatting.
- Support for both `pdfplumber` and `pdfminer` libraries.
- Handles custom page ranges for focused data extraction.
- Output JSON structure with clear title and content separation.
- Optional support for non-English text (e.g., Hindi).

## Requirements

Make sure you have the following libraries installed:

- `pdfplumber`
- `pdfminer.six`
- `json`

Install them using:

```bash
pip install pdfplumber pdfminer.six
```

## Usage

### Step 1: Extract Largest Font Size
Identify the largest font size (excluding digits) in a specified range of pages:

```python
import pdfplumber

path = "example.pdf"

def is_digit(char):
    return char.isdigit()

def extract_font_info(page):
    max_font_size = 0
    for element in page.chars:
        current_char = element['text']
        if is_digit(current_char):
            continue
        current_font_size = element['size']
        if current_font_size > max_font_size:
            max_font_size = current_font_size
    return max_font_size

with pdfplumber.open(path) as pdf:
    total_pages = len(pdf.pages)
    start_page = int(total_pages * 0.5)
    end_page = int(total_pages * 0.6)

    largest_font_size = 0
    for page_number in range(start_page, end_page):
        page = pdf.pages[page_number]
        current_page_max_font_size = extract_font_info(page)
        if current_page_max_font_size > largest_font_size:
            largest_font_size = current_page_max_font_size

print(f"Largest Font Size (excluding digits): {largest_font_size}")
```

### Step 2: Extract Data into JSON
Once the largest font size is identified, extract text data into JSON format:

#### Using `pdfplumber`:

```python
import pdfplumber
import json

def extract_text_from_pdf(path, largest_font_size):
    temp_pdf = []
    temp_dict = {"title": "", "data": ""}

    with pdfplumber.open(path) as pdf:
        for page in pdf.pages:
            temp_title, page_data = extract_page_data(page.chars, largest_font_size)
            if temp_title:
                if len(temp_dict["data"]) > 0:
                    temp_pdf.append(temp_dict)
                temp_dict = {"title": temp_title, "data": page_data}
            else:
                temp_dict["data"] += page_data
        if len(temp_dict["data"]) > 0:
            temp_pdf.append(temp_dict)

    return temp_pdf

def extract_page_data(chars, largest_font_size):
    temp_title = ""
    page_data = ""
    for span in chars:
        if span['size'] >= largest_font_size:
            temp_title += span['text']
        else:
            page_data += span['text']
    return temp_title, page_data

def main():
    path = "example.pdf"
    largest_font_size = 24  # Replace with calculated value
    temp_pdf = extract_text_from_pdf(path, largest_font_size)
    with open(f"{path}_output.json", "w", encoding='utf-8') as out_file:
        json.dump(temp_pdf, out_file, indent=6)

if __name__ == "__main__":
    main()
```

#### Using `pdfminer`:

```python
from pdfminer.high_level import extract_pages
from pdfminer.layout import LTTextContainer, LTChar
import json

def extract_text_with_pdfminer(path, largest_font_size):
    temp_pdf = []
    page_data = ""
    temp_dict = {}

    for page_layout in extract_pages(path):
        for element in page_layout:
            if isinstance(element, LTTextContainer):
                for text_line in element:
                    for chr in text_line:
                        if isinstance(chr, LTChar):
                            if chr.size >= largest_font_size:
                                if len(page_data) > 0:
                                    temp_dict['data'] = page_data
                                    temp_pdf.append(temp_dict)
                                    temp_dict = {}
                                temp_dict["title"] = text_line.get_text()
                                page_data = ""
                            else:
                                page_data += text_line.get_text()

    with open(f"{path}_output.json", "w", encoding='utf-8') as out_file:
        json.dump(temp_pdf, out_file, indent=6, ensure_ascii=False)

path = "example.pdf"
largest_font_size = 24  # Replace with calculated value
extract_text_with_pdfminer(path, largest_font_size)
```

## Performance

The scripts are optimized for large PDFs by focusing on specific page ranges and excluding numeric characters during font analysis.

## Output

The output JSON format:

```json
[
    {
        "title": "Title Extracted",
        "data": "Associated content"
    },
    {
        "title": "Another Title",
        "data": "More content"
    }
]
```


Flow chart Image
![Flow chart](https://github.com/sagarbangade/PDF-to-json-PDFminer/assets/109343765/1372c013-4e8a-4b9c-bdb0-b06182b1499a)


## Contributing

Contributions are welcome! Feel free to submit issues or pull requests.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.


