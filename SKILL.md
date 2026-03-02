---
name: excel-report
description: "엑셀 보고서 작성 스킬. 사용자가 '엑셀 보고서', '엑셀 보고서 만들어줘', '보고서 엑셀로', 'xlsx 보고서' 등을 요청할 때 자동으로 적용된다. A4 용지 규격에 맞춘 인쇄 설정, 눈금선 제거, 깔끔한 보고서 포맷을 기본으로 적용한다."
---

# 엑셀 보고서 스킬

## 반드시 적용되는 기본 규칙

"엑셀 보고서"를 만들 때 아래 설정은 **무조건 적용**한다. 사용자가 별도로 지시하지 않아도 항상 포함.

### 1. 눈금선(Gridlines) 완전 제거
```python
# 화면 표시 눈금선 제거
sheet.sheet_view.showGridLines = False

# 인쇄 시 눈금선도 제거
sheet.sheet_properties.pageSetUpPr = openpyxl.worksheet.properties.PageSetupProperties(fitToPage=False)
sheet.print_options = openpyxl.worksheet.page.PrintOptions(gridLines=False)
```

### 2. A4 용지 인쇄 설정
```python
from openpyxl.worksheet.page import PageMargins

# A4 용지 설정
sheet.page_setup.paperSize = sheet.PAPERSIZE_A4  # A4 = 9
sheet.page_setup.orientation = 'portrait'         # 세로 방향 (가로: 'landscape')
sheet.page_setup.fitToWidth = 1                   # 가로 1페이지에 맞춤
sheet.page_setup.fitToHeight = 0                  # 세로는 자동 (내용에 따라)
sheet.sheet_properties.pageSetUpPr = openpyxl.worksheet.properties.PageSetupProperties(fitToPage=True)

# 여백 설정 (인치 단위, A4 표준 여백)
sheet.page_margins = PageMargins(
    left=0.7,
    right=0.7,
    top=0.75,
    bottom=0.75,
    header=0.3,
    footer=0.3
)

# 인쇄 영역 가운데 정렬
sheet.print_options.horizontalCentered = True
```

### 3. 인쇄 영역 설정
```python
# 데이터가 있는 영역까지 자동 설정
from openpyxl.utils import get_column_letter
max_col_letter = get_column_letter(sheet.max_column)
sheet.print_area = f'A1:{max_col_letter}{sheet.max_row}'

# 첫 행 반복 인쇄 (헤더가 있는 경우)
sheet.print_title_rows = '1:1'
```

## 보고서 기본 스타일

### 폰트
```python
from openpyxl.styles import Font, Alignment, Border, Side, PatternFill

# 기본 폰트: 맑은 고딕 10pt
default_font = Font(name='맑은 고딕', size=10)

# 제목: 맑은 고딕 14pt 굵게
title_font = Font(name='맑은 고딕', size=14, bold=True)

# 소제목: 맑은 고딕 11pt 굵게
subtitle_font = Font(name='맑은 고딕', size=11, bold=True)

# 헤더(표 제목행): 맑은 고딕 10pt 굵게 + 흰색
header_font = Font(name='맑은 고딕', size=10, bold=True, color='FFFFFF')
```

### 표 헤더 스타일
```python
# 헤더 배경: 진한 남색
header_fill = PatternFill('solid', fgColor='2B3A67')

# 헤더 정렬: 가운데
header_alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
```

### 테두리
```python
# 얇은 테두리 (표 안쪽)
thin_border = Border(
    left=Side(style='thin', color='D0D0D0'),
    right=Side(style='thin', color='D0D0D0'),
    top=Side(style='thin', color='D0D0D0'),
    bottom=Side(style='thin', color='D0D0D0')
)

# 헤더 하단 굵은 선
header_bottom = Border(
    bottom=Side(style='medium', color='2B3A67')
)
```

### 교차 행 색상 (Zebra Stripe)
```python
# 짝수 행 연한 회색 배경
even_row_fill = PatternFill('solid', fgColor='F5F5F5')
```

### 숫자 형식
```python
# 금액: 천 단위 쉼표
number_format_won = '#,##0'           # 1,234,567
number_format_won_unit = '#,##0"원"'  # 1,234,567원

# 퍼센트
number_format_pct = '0.0%'           # 12.3%

# 날짜
number_format_date = 'YYYY-MM-DD'    # 2025-03-02
```

## 전체 적용 템플릿 코드

아래는 "엑셀 보고서"를 만들 때 항상 사용하는 기본 뼈대이다:

```python
import openpyxl
from openpyxl import Workbook
from openpyxl.styles import Font, Alignment, Border, Side, PatternFill
from openpyxl.utils import get_column_letter
from openpyxl.worksheet.page import PageMargins
import openpyxl.worksheet.properties

def create_report_workbook():
    wb = Workbook()
    return wb

def apply_report_format(sheet, title_row=1, header_row=2, data_start_row=3):
    """모든 엑셀 보고서에 공통 적용하는 포맷"""
    
    # ═══ 1. 눈금선 제거 ═══
    sheet.sheet_view.showGridLines = False
    
    # ═══ 2. A4 인쇄 설정 ═══
    sheet.page_setup.paperSize = sheet.PAPERSIZE_A4
    sheet.page_setup.orientation = 'portrait'
    sheet.page_setup.fitToWidth = 1
    sheet.page_setup.fitToHeight = 0
    sheet.sheet_properties.pageSetUpPr = openpyxl.worksheet.properties.PageSetupProperties(fitToPage=True)
    
    # 여백
    sheet.page_margins = PageMargins(
        left=0.7, right=0.7,
        top=0.75, bottom=0.75,
        header=0.3, footer=0.3
    )
    sheet.print_options.horizontalCentered = True
    
    # ═══ 3. 인쇄 영역 ═══
    if sheet.max_column and sheet.max_row:
        max_col = get_column_letter(sheet.max_column)
        sheet.print_area = f'A1:{max_col}{sheet.max_row}'
    
    # ═══ 4. 헤더 행 반복 인쇄 ═══
    sheet.print_title_rows = f'{header_row}:{header_row}'
    
    # ═══ 5. 기본 폰트 적용 ═══
    for row in sheet.iter_rows(min_row=1, max_row=sheet.max_row, max_col=sheet.max_column):
        for cell in row:
            if cell.value is not None:
                cell.font = Font(name='맑은 고딕', size=10)
                cell.alignment = Alignment(vertical='center')
    
    # ═══ 6. 헤더 스타일 ═══
    for cell in sheet[header_row]:
        if cell.value is not None:
            cell.font = Font(name='맑은 고딕', size=10, bold=True, color='FFFFFF')
            cell.fill = PatternFill('solid', fgColor='2B3A67')
            cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
    
    # ═══ 7. 데이터 행 교차 색상 + 테두리 ═══
    thin_border = Border(
        left=Side(style='thin', color='D0D0D0'),
        right=Side(style='thin', color='D0D0D0'),
        top=Side(style='thin', color='D0D0D0'),
        bottom=Side(style='thin', color='D0D0D0')
    )
    even_fill = PatternFill('solid', fgColor='F5F5F5')
    
    for row_idx in range(data_start_row, sheet.max_row + 1):
        for cell in sheet[row_idx]:
            cell.border = thin_border
            if row_idx % 2 == 0:
                cell.fill = even_fill
    
    # ═══ 8. 열 너비 자동 조정 ═══
    for col_idx in range(1, sheet.max_column + 1):
        max_length = 0
        col_letter = get_column_letter(col_idx)
        for row in sheet.iter_rows(min_col=col_idx, max_col=col_idx):
            for cell in row:
                if cell.value:
                    cell_length = len(str(cell.value))
                    if cell_length > max_length:
                        max_length = cell_length
        adjusted_width = min(max(max_length + 4, 10), 40)
        sheet.column_dimensions[col_letter].width = adjusted_width
    
    return sheet
```

## 사용 흐름

1. 데이터를 시트에 입력
2. `apply_report_format(sheet)` 호출하여 보고서 포맷 일괄 적용
3. 추가 커스텀 스타일 적용 (필요시)
4. 저장 후 `scripts/recalc.py`로 수식 재계산 (수식이 있는 경우)

## 주의사항
- 이 스킬은 /mnt/skills/public/xlsx/SKILL.md 의 기본 xlsx 스킬과 함께 사용한다
- 기본 xlsx 스킬의 수식 규칙, 에러 검증 규칙을 모두 따른다
- "엑셀 보고서"가 아닌 단순 데이터 파일은 이 스킬을 적용하지 않는다
