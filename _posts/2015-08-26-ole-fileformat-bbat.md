---
layout: post
title: OLE 파일 포맷 (2) - 헤더 블록
description:  OLE 파일의 (-1)블록인 헤더 블록에 대해 알아본다.
date: 2015-08-26 10:58:03
tags: OLE 파일포맷 파이썬
---

헤더 블록은 OLE 블록 중 -1 블록에 해당하는 영역이다. 이 영역에는 OLE 파일 전체의 구조를 접근하기 위한 주요 정보들이 존재한다. 자세한 값을 살펴보기 전에 아래의 소스 코드를 입력하여 해당 내용을 비교하면서 보면 좋을 것이다.

[소스 코드 : test_ole_block.py]

```
001 : # -*- coding:utf-8 -*-
002 : # Module : test_ole_block.py
003 :
004 : import ole
005 : import hexdump
006 : import sys
007 :
008 : #-----------------------------------------------------------
009 : # Test 코드
010 : # 사용법 :
011 : # test_ole_block.py [OLE 블록번호] [출력 위치] [출력 크기]
012 : #-----------------------------------------------------------
013 : # 인자값 검사
014 : if len ( sys.argv ) != 4 :
015 :      print "Input value error!"
016 :      exit()
017 :
018 : # 특정 블록을 읽어 buffer에 저장
019 : fp = open ( "test_ole.hwp", "rb" )
020 : Buffer = ole.ReadBlock ( fp, int(sys.argv[1]) )
021 : fp.close()
022 :
023 : # Buffer를 Hex 덤프
024 : hexdump.Buffer ( Buffer, int(sys.argv[2]), int(sys.argv[3]))
```

[소스 코드 설명]

* 4 ~ 6행 : 필요한 모듈을 import 한다.
* 13 ~ 16행 : 인자 값을 검사한다. 만약 4개의 인자값이 입력되지 않을 경우 에러 처리를 하고 종료한다.
* 18 ~ 21행 : test_ole.hwp 파일을 열어 주어진 인자값을 통해 OLE의 특정 블록을 한 블록 읽는다. 
* 23 ~ 24행 : 읽어 들인 한 개의 블록을 Hex 덤프로 출력한다. 이때 인자 값으로 주어진 출력 위치, 출력 크기에 맞게 출력을 조절한다.

아래의 그림은 test_ole_block.py를 통해 값을 출력할 OLE 파일(test_ole.hwp)의 헤더 블록을 HexCmp [^1] 프로그램을 통해 읽어 들인 모습이다.

![](/images/2015/17756E34-221F-455A-A86E-6A38FDF176B9.png)

## 주요 항목

이제 헤더 블록의 주요 항목들에 대해서 살펴보자.

![](/images/2015/ole_header_1.png)

아래와 같이 시작위치와 크기를 입력하면 결과를 확인할 수 있다.

![](/images/2015/4D58D9F8-7D32-40AB-91B2-E7CCAC1D2ACD.png)

결과에서 볼 수 있듯이 일반적인 값(D0 CF 11 E0 A1 B1 1A E1)과 동일한 결과 값이 출력되었으므로 이 파일은 OLE 파일이며, 그 중 헤더 블록이 맞음을 의미한다.

![](/images/2015/ole_header_2.png)

아래의 실행 결과를 보면 -1 블록의 44 위치에 4 Byte 값은 08 00 00 00 (역워드 값으로 전환하면 8) 이다. 

![](/images/2015/c1da0bed38b35f06bec826ef08086e9f.png)

즉 8개의 BBAT Depot를 가지고 있다는 의미이다. 이 값이 크면 클수록 실제 OLE 파일의 크기도 커진다. 저장소가 많아진다는 것은 블록의 숫자도 많다는 의미이기 때문이다.

![](/images/2015/ole_header_3.png)

아래의 실행 결과를 보면 -1 블록의 48 위치에 4 Byte 값은 02 00 00 00 (역워드 값으로 전환하면 2) 이다. 

![](/images/2015/D9435007-94BC-46C0-A546-F664741BBD65.png)

즉, 2 블록이 프로퍼티(Property) 역역의 시작 위치라는 의미이다. 그렇다면 2 블록을 Hex 덤프 해보자. 아래 결과는 2 블록의 0 위치에서 128 Byte 내용을 Hex 덤프한 것이다.

![](/images/2015/41E7BB9F-AC25-45BE-9E87-569752FE9660.png)

위 결과를 보면 읽을 수 있는 텍스트 결과 값 중에 Root Entry 라는 내용을 우측 ASCII 영역에서 볼 수 있다. 이 값은 나중에 다루게 될 프로퍼티 영역의 시작을 알리는 문자열이다. 

![](/images/2015/ole_header_4.png)

아래의 실행 결과를 보면 -1 블록의 60 위치에 4 Byte 값은 07 00 00 00 (역워드 값으로 전환하면 7) 이다. 

![](/images/2015/25440E9F-222D-499E-B0AB-C2D3B0D51569.png)

즉, 7 블록은 SBAT 정보 역역의 시작 위치라는 의미이다. SBAT에 대해서는 4.2.6절에서 구체적으로 볼 것이다.

![](/images/2015/ole_header_5.png)

아래의 실행 결과를 보면 -1 블록의 64 위치에 4 Byte 값은 01 00 00 00 (역워드 값으로 전환하면 1) 이다. 

![](/images/2015/26026EB7-EF54-43B2-803C-E8A238EF8DEA.png)

즉, 1개의 SBAT Depot을 가지고 있다는 의미가 된다.

![](/images/2015/ole_header_6.png)

아래의 실행 결과를 보면 -1 블록의 76 위치에서 임의적으로 52 Byte 만큼 읽어 출력하였다. 사실 아래의 내용은 4 Byte 씩  배열로 구성되어 있다. 따라서 실제 존재하는 값은 역워드 값으로 전환하면 0x3, 0x77, 0x78, 0x79, 0x7A, 0x25B, 0x25C, 0x25D 이다. 나머지 0xFFFFFFFF 값은 의미 없는 값이다.

![](/images/2015/BB5F40D8-89A4-4020-A609-7EC99BB6AD77.png)

이미 앞에서 아래의 결과값을 본 적이 있다.

![](/images/2015/c1da0bed38b35f06bec826ef08086e9f.png)

바로 Number  of Big Block Allocation Table Depot 값이다. BBAT Depot 정보가 8개(0x3, 0x77, 0x78, 0x79, 0x7A, 0x25B, 0x25C, 0x25D) 있음을 의미한다. 그렇기 때문에 8개의 값 이후의 나머지 0xFFFFFFFF 값은 의미 없는 값들이 된다.

## 실습

앞에서 배운 헤더 블록의 정보를 출력해 주는 프로그램을 만들어 보자.

[소스 코드 : ole_headerblock_info.py] 

[소스 코드 설명]





[^1]: HexCmp : <http://www.fairdell.com/hexcmp/>



***

#### Update

- 2015-08-26 : 최초로 작성