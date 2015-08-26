---
layout: post
title: OLE 파일 포맷 (3) - Block Allocation Table(BBAT)와 BBAT Depot
date: 2015-08-26 13:10:36
tags: OLE 파일포맷 파이썬
---

Big Block Allocation Table(BBAT)은 OLE 내부의 스트림의 위치 정보를 저장하고 있으며, 이들은 모두 링크 구조로 되어있다. 이 BBAT는 OLE 파일이 커지면 커질수록 크기가 증가하게 된다. 여기서 BBAT Depot은 BBAT가 저장된 저장소를 의미한다.

먼저, BBAT Depot을 알아보자. 헤더에는 Number  of Big Block Allocation Table Depot의 값이 존재하며, 이 숫자만큼 Array of Big Block Allocation Table Depot members에 값이 보관되어 있다고 이미 앞에서 설명했다.

![](/images/2015/BB5F40D8-89A4-4020-A609-7EC99BB6AD77.png)

즉, 위 예제에서 보는 바와 같이 8개의 BBAT Depot 정보가 있음을 확인하였다. 그 값이 0x3, 0x77, 0x78, 0x79, 0x7A, 0x25B, 0x25C, 0x25D 이다. 이제 이들 블록을 차례대로 접근하여 512(0x200) Byte씩 읽어 조합한 것이 BBAT이다. 

![](/images/2015/76DB1967-6D28-4E10-91A2-AAEC95A44841.png)

아래는 3번 블록을 읽어 화면에 출력한 결과이다.

![](/images/2015/566DBD50-A7E7-4690-99F8-03188F7338EF.png)

위 결과를 아래와 같이 4Byte씩 끊으면 의미 있는 데이터가 된다. FAT32 파일 시스템에서 File Allocation Table[^1]의 모습과 유사하다. 아래의 그림은 Big Block Allocation Table(BBAT)의 각 Entry의 세부적인 구조를 표현한 것이다.

![](/images/2015/A7431D8B-D1BA-450D-A39F-FD37A78F7539.png)

아래의 그림은 3 블록을 읽은 결과 값을 BBAT의 Entry 구조에 맵핑한 것이다.

![](/images/2015/0709B690-F701-49D0-B557-F5A99C473E3B.png)

헤더 블록에서 48 위치에 4 Byte 값으로 저장된 Start block of Property를 확인하였다. 다시 아래에 그 결과 값을 보면 역워드로 전환한 값이 2임을 알 수 있다.

![](/images/2015/D9435007-94BC-46C0-A546-F664741BBD65.png)

즉, 프로퍼티의 시작이 2 블록부터라는 것을 알 수 있다. 2 블록 다음은 어느 블록일까? 바로 Entry 2에 저장된 값을 확인하면 그 다음 블록을 알 수 있다. [그림 9]에 의하면 Entry 2에 저장되어 있는 값은 0x00000004이다. 따라서 2 블록 다음은 4 블록이라는 의미이다. 이제 Entry 4의 값을 보자. 0x00000005이다. Entry 5에는 0x00000006이, Entry 6에는 0x00000012가 저장되어 있다. 0x00000012는 10진수로 18이므로 Entry 18을 확인하면 0xFFFFFFFE가 있다. 

즉, 아래와 같은 Entry를 차례로 참조하게 되며, 이를 chain이라 한다. 이 chain에 등장하는 Entry 값들은 결국 읽어야 할 각 블록의 번호와 일치한다.

```
Entry 2 → Entry 4 → Entry 5 → Entry 6 → Entry 18
```

2 블록, 4 블록, 5 블록, 6 블록, 18 블록을 차례로 512 Byte 크기만큼 읽어서 합치면 하나의 스트림이 완성된다. 참고로 모든 BBAT에 존재하는 모든 블록 값들은 무조건 512Byte로 읽어야 한다. 

Entry에는 아래의 값들이 올 수 있다.

![](/images/2015/ole_header_7.png)

## 실습

Big Block Allocation Table(BBAT) Depot의 정보를 조합하여 BBAT를 구성하고, 이후 Start block of Property 값을 헤더 블록에서 읽어 프로퍼티를 구성하는 블록의 링크들을 출력해주는 프로그램을 만들어 보자. 예상대로라면 아래의 chain 정보를 출력해야 한다(단, 적용한 OLE 파일에 따라 그 값은 달라질 수 있다).

```
Entry 2 → Entry 4 → Entry 5 → Entry 6 → Entry 18
```


[소스 코드 : ] 

[소스 코드 설명]

[실행 결과]



[^1]: FILE Allocation Table : http://en.wikipedia.org/wiki/File_Allocation_Table



***

####Update

- 2015-08-26 : 최초로 작성