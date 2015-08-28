---
layout: post
title: OLE 파일 포맷 (4) - 프로퍼티(Property)와 프로퍼티 스토리지(Storage)
description:  OLE 파일 내부에 존재하는 스토리지와 스트림에 대한 정보를 담고 있는 프로퍼티(Property)와 프로퍼티 스토리지(Storage)에 대해 알아본다.
date: 2015-08-27 08:42:43 
tags: OLE 파일포맷 파이썬
---


OLE 파일 내부에는 폴더(스토리지)와 파일(스트림)이 존재하는데 이들 개개의 정보를 프로퍼티라고 하며, 프로퍼티를 보관하는 곳을 프로퍼티(Property) 스토리지라 한다. 아래는 test_ole.hwp 파일의 내부를 SSViewer를 통해 내부를 들여다 본 모습이다. 아래의 좌측에 존재하는 트리구조의 화면이 바로 test_ole.hwp 파일의 스토리지와 스트림의 구성 모양이다. 

![](/images/2015/6F5BE100-B7B1-4E7C-9882-4238E1BEE339.png)

프로퍼티의 트리 구성에 필요한 주요 정보들 담은 블록은 헤더 블록에서 Start block of Property의 값을 통해 그 시작 위치 및 BBAT를 참조하여 프러퍼티를 구성하는 블록 chain을 알아 낸 이후에 chain 정보 순서로 512(0x200) Byte 읽어 합침으로써 만들 수 있다. 우리는 이전 문서[^1]의 실습을 통해 프로퍼티를 구성하는 블록의 chain 정보를 확인하였다.

```
Entry 2 → Entry 4 → Entry 5 → Entry 6 → Entry 18
```

이 Entry의 정보를 블록 정보 및 위치 정보로 변환하면 다음과 같이 된다.

```
2블록(0x0x600) → 4블록(0xA00) → 5블록(0xC00) → 6블록(0xE00) → 18블록(0x2600) 
```

이 위치 정보들을 읽어 합치면 프로퍼티 스토리지가 완성된다. 아래는 해당 블록을 읽어서 출력한 결과이다. 

![](/images/2015/FFA26F8C-C4CD-4E12-A498-4E86F7D1AE45.png)

위의 결과물은 2 블록을 읽은 것이다. 이 곳에 저장된 각 스토리지와 스트림의 정보는 아래와 같다.

* [0]  Root Entry
* [1] FileHeader
* [2] DocInfo
* [3] BodyText

![](/images/2015/0B676672-E5F0-458A-91B3-728999251708.png)

위의 결과물은 4 블록을 읽은 것이다. 이 곳에 저장된 각 스토리지와 스트림의 정보는 아래와 같다.

* [4] HwpSummaryInformation
* [5] BinData
* [6] PrvImage
* [7] PrvText

![](/images/2015/352339FC-800F-43E5-9FC2-C2DDFE9AF363.png)

위의 결과물은 5 블록을 읽은 것이다. 이 곳에 저장된 각 스토리지와 스트림의 정보는 아래와 같다.

* [8] DocOptions
* [9] Scripts
* [10] JScriptVersion
* [11] DefaultJScript

![](/images/2015/1C0D4213-B5F2-417C-B9BC-A3D4D4F6A8A7.png)

위의 결과물은 6 블록을 읽은 것이다. 이 곳에 저장된 각 스토리지와 스트림의 정보는 아래와 같다.

* [12] _LinkDoc
* [13] BIN0001.bmp
* [14] BIN0002.bmp
* [15] BIN0003.bmp

![](/images/2015/9E28BEF0-D450-45D2-B2BD-6E13BE1531B1.png)

위의 결과물은 18 블록을 읽은 것이다. 이 곳에 저장된 각 스토리지와 스트림의 정보는 아래와 같다.

* [16] Section0

지금까지 결과물로 본 스토리지와 스트림의 정보를 SSViewer로 본 test_ole.hwp 내부 구조와 비교해보라. 스토리지와 스트림의 순서는 다르지만 내용은 일치함을 확인할 수 있을 것이다.

## 주요 항목

각 블록에는 최대 4개의 프로퍼티(즉, 스토리지 또는 스트림)의 정보가 저장될 수 있다. 즉, 1개의 프로퍼티 정보는 128(0x80) Byte로 구성이 되어 있다. 여기에서는 2번 블록에 있는 Root Entry 프로퍼티를 기준으로 주요 항목들을 살펴볼 것이다. 

예제의 내용은 프로퍼티 영역의 시작인 2 블록(헤더 블록의 Start block of Property 값을 참조)을 128 Byte 만큼 Hex 덤프하여 표와 비교해서 살펴보도록 하겠다. 

![](/images/2015/ole_pps_1.png)

아래의 실행 결과를 보면 2 블록의 0 위치에 64 Byte 값이 프로퍼티의 이름인데 여기에서는 각 문자 뒤 0x00을 제외하고 읽으면 된다. 따라서 Root Entry가 된다.

![](/images/2015/2b36e4d7e50abfc6922fae20fc3e80ec1.png)


![](/images/2015/ole_pps_2.png)

아래의 실행 결과를 보면 2 블록의 65 위치에 2 Byte 값은 16 00(역워드 값으로 전환하면 0x16)이다. 

![](/images/2015/2b36e4d7e50abfc6922fae20fc3e80ec.png)

Root Entry는 문자열의 길이가 NULL 문자를 포함하면 11이다. 여기에 한 문자는 2 Byte로 표현하므로 11 * 2 = 22(0x16)가 된다. 

![](/images/2015/ole_pps_3.png)

아래의 실행 결과를 보면 2 블록의 66 위치에 1 Byte 값은 05이다. 

![](/images/2015/2b36e4d7e50abfc6922fae20fc3e80ec2.png)

지금 보고 있는 프로퍼티는 Root Entry 이므로 최상위임을 나타내는 5가 저장되어 있다. 

![](/images/2015/46f9f8f9368ff102bf36d9f94c5a3ebd.png)

위의 결과는 프로퍼티 영역에 하나였던 4 블록에 존재하는 _HwpSummaryInformation의 경우 Type of Property의 값이 2이다. 따라서 이 프로퍼티는 스트림을 의미한다.

![](/images/2015/d2b2bbfeb53fccd168e339c66fc8e578.png)

마찬가지로 위의 결과는 프로퍼티 영역에 하나였던 5 블록에 존재하는 DocOptions의 경우 Type of Property의 값이 1이다. 따라서 이 프로퍼티는 스토리지를 의미한다.

![](/images/2015/ole_pps_4.png)
![](/images/2015/ole_pps_5.png)
![](/images/2015/ole_pps_6.png)

위의 3개의 주요 속성(Previous Property, Next Property, Directory Property)은 각 프로퍼티들의 트리 구조가 어떻게 구성되어 있는지를 나타내는 값들이다. 따라서 이 속성들을 함께 설명한다.

먼저 Root Entry에서의 3개의 속성 값을 보자.

![](/images/2015/2b36e4d7e50abfc6922fae20fc3e80ec3.png)

첫 번째 박스의 값이 Previous Property이며, 여기에서는 0xFFFFFFFF 값을 가진다. 두 번째 박스의 값은 Next Property이며, 여기에서도 0xFFFFFFFF 값을 가진다. 마지막 세 번째 박스의 값은 Directory Property이며, 여기에서는 0x00000003 값을 가진다.

Root Entry는 프로퍼티의 가장 최상위이다. 따라서 이전과 다음 프로퍼티는 존재하지 않고 오로지 하위 프로퍼티만을 가질 뿐이다. 따라서 Directory Property만이 존재하는 것이다. Directory Property 값이 3이므로 3번 프로퍼티가 하위 프로퍼티이다. 이전에 모든 프로퍼티의 이름을 출력했었는데, 그때 프로퍼티 앞에 번호를 부여하였다. 다시 프로퍼티를 정리해 보면 아래와 같다.

* [0]  Root Entry
* [1] FileHeader
* [2] DocInfo
* [3] BodyText
* [4] HwpSummaryInformation
* [5] BinData
* [6] PrvImage
* [7] PrvText
* [8] DocOptions
* [9] Scripts
* [10] JScriptVersion
* [11] DefaultJScript
* [12] _LinkDoc
* [13] BIN0001.bmp
* [14] BIN0002.bmp
* [15] BIN0003.bmp
* [16] Section0

0번 프로퍼티가 Root Entry이며, Root Entry 하위에 존재할 프로퍼티가 3번인 BodyText 프로퍼티이다. Root Entry가 가지는 Previous Property, Next Property, Directory Property 의 속성 값을 가지고 구조를 표현하면 다음과 같다.

![](/images/2015/ole_pps_t1.png)

그렇다면 3번 프로퍼티인 BodyText의 Previous Property, Next Property, Directory Property 속성 값도 확인 해 보자.

![](/images/2015/ff8286fd23f8ff12edfba2f02d7e4bec.png)

BodyText의 Previous Property는 2, Next Property이는 1,, Directory Property는 16(0x10)이다. 이 구조를 그림으로 표현하면 아래와 같다. 

![](/images/2015/ole_pps_t2.png)

![](/images/2015/ole_pps_7.png)

OLE 파일은 크게 헤더 블록과 데이터 블록으로 구성되는데, 그중 데이터 블럭의 대부분은 스트림의 데이터로 채워져 있다. 이 데이터들은 흩어져 있기 때문에 특정 스트림이 어떤 블록에 저장 되어 있는지의 정보가 필요하며 이는 Big/Small Block Allocation Table에 링크 정보로 저장되어 있다. 프로퍼티의 정보를 저장한 이 영역에는 해당 스트림 데이터 블록의 시작 블록 값만 저장하고 있다. 따라서 이 값은 프로퍼티의 타입이 스트림일 때 값이 세팅되어 있다. 아래의 HwpSummaryInformation 프로퍼티(타입이 스트림이다)의 경우를 보여주고 있다.

![](/images/2015/46f9f8f9368ff102bf36d9f94c5a3ebd1.png)

만약 프로퍼티의 타입이 스토리지인 경우에는 0이 저장 되어 있다. 아래는 DocOptions 프로퍼티(타입이 스토리지이다)의 경우를 보여주고 있다.

![](/images/2015/1d2b2bbfeb53fccd168e339c66fc8e578.png)

![](/images/2015/ole_pps_8.png)

프로퍼티의 타입이 스트림이라면 데이터 블록의 영역을 차지하고 있으므로 데이터의 크기가 필요하다. 여기에서 중요한 점은 크기가 4096(0x1000) Byte 크기보다 크거나 같다면 해당 프로퍼티는 Big Block Allocation Table(BBAT)을 참조하여 링크 구조를 생성한다. 아래의 Section0 프로퍼티의 크기는 0x751F로 0x1000보다는 크다. 

![](/images/2015/9C166AC2-244D-4EBD-AD57-AC375F1DE414.png)

따라서 Section0는 BBAT를 참조하여 링크 구조를 생성해야 한다. 즉, Section0 프로퍼티의 Starting block of Property의 값은 0x3BE이다.

![](/images/2015/123Image.png)

그러므로 BBAT의 Entry 0x3BE에 존재하는 값을 참조하여 다음 Entry를 수집하여 0xFFFFFFFE 값을 가지는 Entry까지의 링크 구조를 완성해야 한다. 그 Entry 번호를 모두 수집하여 각 블록에서 512(0x200) Byte씩 읽어 합친다. 마지막에는 Section0의 프로퍼티 크기만큼 크기를 조절하면 된다.

만약 프로퍼티의 크기가 4096(0x1000) Byte 보다 작다면 해당 프로퍼티는 Small Block Allocation Table을 참조하여 링크 구조를 생성하게 된다. 앞서 설명한 Section0 프로퍼티와는 달리 HwpSummaryInformation 프로퍼티는 크기가 0x215이므로 0x1000보다는 작다.

![](/images/2015/hwpsinfor.png)

따라서 HwpSummaryInformation  프로퍼티는 Small Block Allocation Table을 참조하여 링크 구조를 생성해야 하며, 이렇게 취합된 블록을 HwpSummaryInformation 프로퍼티 크기만큼 크기를 조절하면 된다. Small Block Allocation Table에 대해서는 다음에 자세히 다루도록 한다.


## 실습

[소스 코드 : ] 

[소스 코드 설명]

[실행 결과]



[^1]: 이전링크 : [OLE 파일 포맷 (3) - Block Allocation Table(BBAT)와 BBAT Depot](/ole-fileformat-header/)



***

####Update

- 2015-08-27 : 최초로 작성