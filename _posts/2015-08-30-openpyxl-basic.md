---
layout: post
title: 파이썬으로 엑셀 보고서 만들기
description:  프로그래밍을 하다보면 엑셀 보고서를 자동으로 생성해야 할 경우가 있다. 그래서 파이썬의 외부 모듈인 openpyxl을 이용하여 엑셀 보고서를 만들어 보기로 한다.
date: 2015-08-30 12:38:20 
tags: 파이썬 엑셀 Openpyxl
---

Openpyxl은 엑셀 파일을 손쉽게 생성할 수 있는 파이썬 외부 라이브러리이다. 가끔 프로그래밍을 하다보면 보고서 출력이 필요한 경우가 있다. 그러면 대체로 생각할 수 있는 방법은 엑셀의 COM 오브젝트를 접근하여 문서를 생성하는 것이다. 그러나 이 방법은 해당 시스템에 엑셀이 설치되어 있어야 하는 단점이 존재한다.

고객이 물어본다.

> "엑셀이 설치되지 않은 시스템에서는 보고서 생성이 안되는건가요?"
> "엑셀 라이선스를 구매해야 한다면 좀 그런데????"

엑셀이 시스템에 설치되지 않아도 할 수 있는 방법을 찾다가 알게 된 Openpyxl은 엑셀이 설치되지 않은 시스템에서도 자유롭게 엑셀 파일이 생성할 수 있다.

<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong>참고</strong></span></legend><!--Creative Commons License--><div style="float: left; width: 88px; margin-top: 3px;"><img alt="Creative Commons License" style="border-width: 0" src="/images/exclamationmark.png"/></div><div style="margin-left: 92px; margin-top: 3px; text-align: justify;">Openpyxl 라이브러리는 아직은 계속적으로 업데이트가 활발히 진행되는 단계로 개발자가 새로운 시도를 많이 하는 스타일로 보여진다. 버전이 바뀔때 마다 예전의 함수들 사용법이 바뀌는 경우가 많아 기존 프로그램을 수정해야 하는 경우가 종종 있다. 본 문서는 <a>Openpyxl 버전 2.2.5를 기준으로 작성 되었음</a>을 명심하기 바란다.
</div></fieldset>


##1. 설치

###1.1 다운로드 및 설치

Openpyxl은 아래의 사이트에서 다운로드 받을 수 있다.

- https://pypi.python.org/pypi/openpyxl/2.2.5

압축을 해제한 후 ```setup.py install```을 입력하면 설치를 할 수 있다. 정상적으로 설치 되었는지의 여부는 이제 파이썬을 구동하여 ```import openpyxl```을 통해 확인할 수 있다.

```
C:\Python27>python.exe
Python 2.7.9 (default, Dec 10 2014, 12:24:55) [MSC v.1500 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import openpyxl
>>> _
```

###1.2 추가 모듈 다운로드 및 설치

Openpyxl이 최신 버전부터 *jdcal*을 필요로 한다. 그래서 추가 라이브러리를 설치해야 한다. 따라서 아래의 사이트에서 jdcal을 다운로드 받는다.

- https://pypi.python.org/pypi/jdcal

압축을 해제한 후 ```setup.py install```을 입력하여 설치한다.

## 2. 문서의 생성하기

이제 Openpyxl이 설치되었으니 간단하게 엑셀 문서를 생성해 보자. 아래의 예제를 실행해보면 test.xlsx 파일이 생성된다. 

```
from openpyxl import Workbook

wb = Workbook()      # 워크북을 생성한다.
ws = wb.active       # 워크 시트를 얻는다.
ws['A1'] = 'Hello'   # A1에 'Hello' 값을 입력한다.
wb.save('test.xlsx') # 엑셀로 저장한다.
```

해당 엑셀을 열어보면  A1 영역에 'Hello'이라는 글자를 입력된 것을 확인할 수 있다.

![](/images/2015/openpyxl/openpyxl_1.png)


## 3. WorkSheet 작업하기

###3.1 Sheet 생성

현재 작업중인 Workbook에 Sheet를 추가할 때에는 create_sheet() 함수를 사용한다. 이때 추가하고자 하는 위치(index)를 지정하고 Sheet의 이름을 입력한다. **만약 한글로 된 Sheet 이름을 입력하고 싶다면 유니코드로 변환하여 입력하면 된다.**

```
# 1번 위치에 'Test Sheet' 이름의 Sheet를 만든다.
ws2 = wb.create_sheet(1, 'Test Sheet') 
ws2['A1'] = 100   # 새로운 Sheet A1에는 숫자 100을 입력한다.
```

![](/images/2015/openpyxl/openpyxl_2.png)

###3.2 Sheet 삭제

불필요한 Sheet는 remove_sheet() 함수를 사용하여 삭제할 수 있다. 인자값으로는 create_worksheet() 함수의 리턴값인 sheet의 핸들을 사용해야 한다.

```
wb.remove_sheet(ws) # ws는 A1에 'Hello'가 있었던 Sheet의 핸들이다.
```

![](/images/2015/openpyxl/openpyxl_3.png)


###3.3 Sheet 이름 변경

Sheet의 이름을 변경하는 방법은 해당 Sheet의 속성 값을 통해서 간단하게 변경할 수 있다.

```
ws2.title = 'New Name' # 'Test Sheet'의 이름을 변경한다.
```

![](/images/2015/openpyxl/openpyxl_4.png)


## 4. Cell 작업하기

### 4.1 Cell 병합

두 개 이상의 Cell을 병합하고 싶을 때가 있다. 이럴 경우 merge_cells() 함수를 사용하면 된다.

```
ws2.merge_cells('A1:E1')  # A1에서 E1까지 Cell을 병합한다.
ws2['A1'] = 'REPORT'      # 병합된 Cell은 A1으로 접근할 수 있다.
```

![](/images/2015/openpyxl/openpyxl_5.png)
  
### 4.2 Cell 폰트 설정 및 정렬

폰트와 정렬을 위해서는 아래처럼 ```import```를 해야 한다.

```
from openpyxl.styles import Font, Alignment
```

이제 각 Cell 마다 폰트를 설정할 수 있다. 이때 폰트의 이름은 한글일 경우 유니코드로 하여 설정해야 한다. 또한 폰트의 크기와 굵기도 정할 수 있다. 

```
ca1 = ws2['A1']

# 폰트 이름은 '맑은 고딕'이고 크기는 15이면서 굵게 속성 설정 
ca1.font = Font(name='맑은 고딕'.decode('cp949'), size=15, bold=True)
```

Cell 내용을 가로, 세로 맞춤을 왼쪽, 오른쪽, 가운데로 정렬할 수 있다.

```
# Cell의 가로, 세로 맞춤을 가운데로 정렬한다.
ca1.alignment = Alignment(horizontal='center', vertical='center')

```

![](/images/2015/openpyxl/openpyxl_6.png)

### 4.3 Cell 테두리

Cell 테두리를 위해 또 다시 새로운 ```import```가 있어야 한다.

```
from openpyxl.styles import Border, Side
```

우선 새로운 Cell에 문자를 입력한다. 

```
# A2에 '테스트' 문자를 입력한다.  
ws2['A2'] = '테스트'.decode('cp949')     
ca2 = ws2['A2']
```

이제 선의 굵기가 가늘고 색깔은 검은색으로 박스 모양을 만들어 Cell에 적용한다.

``` 
# 네모 박스를 설정한다.
box = Border(left=Side(border_style="thin", 
                   color='FF000000'),
         right=Side(border_style="thin",
                    color='FF000000'),
         top=Side(border_style="thin",
                  color='FF000000'),
         bottom=Side(border_style="thin",
                     color='FF000000'),
         diagonal=Side(border_style="thin",
                       color='FF000000'),
         diagonal_direction=0,
         outline=Side(border_style="thin",
                      color='FF000000'),
         vertical=Side(border_style="thin",
                       color='FF000000'),
         horizontal=Side(border_style="thin",
                        color='FF000000')
        )

ca2.border = box # Cell 테두리를 적용한다.
```

![](/images/2015/openpyxl/openpyxl_7.png)


### 4.4 Cell 색상

Cell에 색상 값을 적용하기 위해서는 별도의 ```import```를 해야 한다.

```
from openpyxl.styles import PatternFill, Color
```

```import```가 완료되었다면 이제 원하는 Cell에 색상 값을 적용할 수 있다.

```
ca2.fill = PatternFill(patternType='solid', fgColor=Color('FFC000'))
```

![](/images/2015/openpyxl/openpyxl_8.png)


### 4.5 Cell 고정

엑셀로 보고서를 만들게 되면 특정 역역은 스크롤되지 않고 고정된 형태로 있기를 원하는 경우가 있다. 그러기 위해서는 다음과 같이 할 수 있다.

```
ws2.freeze_panes = 'A2' # A2 위쪽을 고정
```

![](/images/2015/openpyxl/openpyxl_9.png)


## 5. 결론

프로그램에서 엑셀로 보고서를 생성해야 하는 경우 엑셀이 설치되어야지만 생성 가능한 경우가 많다. 하지만 모든 시스템에 엑셀이 설치되었다는 보장을 하기가 어렵다. 

따라서 엑셀 설치와 무관하게 동작하기 위해서는 파이썬 개발자라면 라이브러리가 있으면 Openpyxl을 고려해보는 것도 좋을 것이다.


***

####Update

- 2015-08-30 : 최초로 작성