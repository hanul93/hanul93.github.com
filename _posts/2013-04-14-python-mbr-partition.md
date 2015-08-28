---
layout: post

title: 파이썬을 이용한 파티션 테이블 분석
description: 포렌식을 위해서 파일시스템 분석은 필수적이다. 그 중에서 FAT 파일시스템의 파티션 테이블을 분석하는 방법에 대해 알아본다.

date: 2013-04-14 20:29:15
tags: 파이썬 포렌식
---


파이썬을 이용하여 자신만의 분석도구를 만드는 것은 상당히 중요하다. 그 이유는 어떤 포렌식 도구보다도 향후 필요한 기능을 스스로 추가하고 확장하기 용이하기 때문이다. 만약 자신이 즐겨 사용하는 포렌식 도구가 특정 기능을 지원해주지 않는다면 결국 해당 기능을 가진 다른 포렌식 도구를 찾아야 하기 때문이다. 

여기에서는 파일시스템 포렌식 도구에서 가장 기본이 되는 하드디스크의 파티션 테이블 분석 도구를 파이썬을 이용 해서 개발해 본다. 

##1. 기본적인 파이썬 코드 
기본적으로 파이썬을 이용하여 파일을 다루는 프로그램을 작성해본다. 

{% highlight python %}
# C드라이브 루트에 존재하는 1.txt 파일을 이진 읽기 모드로 파일을 연다 
handle = open('c:\\1.txt', 'rb') 

# 파일 전체를 읽어 버퍼에 담는다 
buf = handle.read() 

# 파일을 닫는다 
handle.close()
{% endhighlight %}


위의 프로그램은 파일을 열고 읽는 간단한 기능을 작성해 본 것이다. 특정 위치를 이동해서 파일의 특수 영역을 읽을 수도 있다.

{% highlight python %}
# C드라이브 루트에 존재하는 1.txt 파일을 이진 읽기 모드로 파일을 연다 
handle = open('c:\\1.txt', 'rb') 

# 파일의 맨 앞으로 이동한다 
handle.seek(0) 

# 512 Byte를 읽어 버퍼에 담는다 
buf = handle.read(512) 

# 파일을 닫는다 
handle.close()
{% endhighlight %}

##2. 하드디스크 MBR 읽기 

이제 하드디스크의 MBR을 읽어보자. 윈도우 운영체제에서는 하드디스크도 파일처럼 핸들링할 수 있다. 이전 소스코드에서 open 함수의 인자값을 다음과 같이 하면 된다. 

{% highlight python %}
# HDD1을 이진 읽기 모드로 연다 
handle = open('\\\\.\\PhysicalDrive1', 'rb') 

# 하드디스크의 맨 앞으로 이동한다 
handle.seek(0) 

# 512 Byte를 읽어 버퍼에 담는다 
mbr = handle.read(512) 

# 하드디스크를 닫는다 
handle.close()
{% endhighlight %}

만약 HDD0을 접근하고 싶다면 open 함수의 인자값만 다음과 같이 바꾸면 된다.

```
handle = open('\\\\.\\PhysicalDrive0', 'rb')
```

MBR이 정확하게 읽혀진 것이 맞는지를 체크하기 위해 MBR의 Magic ID도 체크하도록 기능을 추가하였다.

{% highlight python %}
# MBR의 Magic ID가 맞나? 
if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA : 
    print 'MBR Read Success' 
else : 
    print 'MBR Read Fail' 
{% endhighlight %}

##3. 파티션 테이블 읽기 
우리가 접근하려는 HDD1은 다음과 같은 파티션 구조를 가지고 있다. 

![](/images/2014/part.jpg)

파티션 테이블은 MBR 코드에서 0x1BE 위치에서 0x10씩 4개가 존재한다. 따라서 다음과 같이 그 정보를 추출해 낼 수 있다.

{% highlight python %}
# MBR의 Magic ID가 맞나? 
if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA : 
    print 'MBR Read Success' 
    
    # 파티션 정보 획득 
    part = [] 

    for i in range(4) : 
        part.append(mbr[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10]) 
else : 
    print 'MBR Read Fail' 
{% endhighlight %}

이제 파티션 테이블에 존재하는 4개의 파티션 정보가 part 변수에 들어가 있다. 이제 이 part 변수를 확인하여 어떤 파티션 정보가 존재하는지 확인해보자.

{% highlight python %}
# MBR의 Magic ID가 맞나? 
if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA : 
    print 'MBR Read Success' 
    
    # 파티션 정보 획득 
    part = [] 

    for i in range(4) : 
        part.append(mbr[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])

    # 각 파티션의 시스템 타입만 출력 
    for i in range(4) : 
        p = part[i] 
        if ord(p[4]) == 0xF or ord(p[4]) == 0x5 : 
            print '%d : ExtendedPartition' % i # 확장 파티션 
        elif ord(p[4]) != 0 : 
            print '%d : PrimaryPartition' % i # 주 파티션
else : 
    print 'MBR Read Fail' 
{% endhighlight %}

여기까지 실행결과는 다음과 같다.

````
MBR Read Success 
0 : PrimaryPartition 
1 : ExtendedPartition

````

이제 본격적으로 주 파티션과 확장 파티션을 처리할 것이므로 main 영역이 계속적으로 확장될 것이므로 함수로 별도로 분리하여 처리하기 쉽게 한다.

{% highlight python %}
# 확장 파티션 처리 
def ExtendedPartition(part, start) : 
    print 'ExtendedPartition' 
    
# 주 파티션 처리 
def PrimaryPartition(part) : 
    print 'PrimaryPartition'

if __name__ == '__main__' : 
    # HDD1를 연다 
    handle = open('\\\\.\\PhysicalDrive1', 'rb') 
    
    # MBR (0번 섹터) 위치로 이동한다 
    handle.seek(0 * 512) # Offset이므로 512를 곱함 
    
    # MBR 역역을 읽는다 
    mbr = handle.read(0x200) 
    
    # MBR의 Magic ID가 맞나? 
    if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA : 
        print 'MBR Read Success' 
        
        # 파티션 정보 획득 
        part = [] 

        for i in range(4) : 
            part.append(mbr[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])

        # 각 파티션의 시스템 타입만 출력 
        for i in range(4) : 
            p = part[i] 
            if ord(p[4]) == 0xF or ord(p[4]) == 0x5 : 
                ExtendedPartition(p) # 확장 파티션 처리
            elif ord(p[4]) != 0 : 
                PrimaryPartition(p) # 주 파티션 처리
    else : 
        print 'MBR Read Fail' 
        
    # HDD0를 닫는다 
    handle.close()
{% endhighlight %}

##4. 주 타피션 처리하기 
주 파티션에서 중요 정보를 추출하여 출력해보자. 파이썬에서는 바이너리를 정수형으로 변환하기 위해서는 struct 모듈을 이용하여 처리한다. 아래 예제는 Starting LBA 값을 추출하여 출력한 것이다. 

{% highlight python %}
import struct 

# 주 파티션 처리 
def PrimaryPartition(part) : 
    print 'PrimaryPartition' 
    
    # Starting LBA 
    start = struct.unpack('<L', part[ 8: 8+4])[0] 
    print start
{% endhighlight %}

실행결과는 다음과 같다. 

```
MBR Read Success
PrimaryPartition
63
ExtendedPartition
```

이제 주 파티션의 용량도 확인해보자. 파티션 정보에서 오프셋 12 위치에 파티션의 섹터 수가 기록되어 있다. 따라서 용량을 구하려면 한 섹터를 구성하고 있는 Byte 수를 곱해서 1024를 3번 나누면 GByte로 표현할 수 있다.

{% highlight python %}
import struct 

# 주 파티션 처리 
def PrimaryPartition(part) : 
    print 'PrimaryPartition' 
    
    # Starting LBA 
    start = struct.unpack('<L', part[ 8: 8+4])[0] 
    print start

    # Num of Sector 
    num = struct.unpack('<L', part[12:12+4])[0] 
    size = (num * 512.) / 1024 / 1024 / 1024 
    print size
{% endhighlight %}

실행결과를 보자. 

```
MBR Read Success
PrimaryPartition
63
0.099555015564
ExtendedPartition
```

사실 주 파티션에는 대략 102MByte가 설정되어 있기 때문에 GByte로 표현하다 보니 소수점 형태로 출력이 되었다. 그럼 섹터를 입력하면 용량을 계산해서 문자열로 리턴하는 함수를 하나 추가해 보자. 

{% highlight python %}
# 섹터를 용량으로 변환하여 문자열로 리턴한다 
def StringCalcSector2Size(num) : 
    str_size = ['B', 'KB', 'MB', 'GB'] 
    
    t = size = (num * 512.) 
    
    for i in range(4) : 
        t = t / 1024 # 한번 1024를 나누어 
        if int(t) > 0 : # 정수 값이 있는지를 체크 
            size = size / 1024 
        else : 
            break # 정수 값이 없다면 종료 
            
    return '%.2f %s' % (size, str_size[i])

# 주 파티션 처리 
def PrimaryPartition(part) : 
    print 'PrimaryPartition' 
    
    # Starting LBA 
    start = struct.unpack('<L', part[ 8: 8+4])[0] 
    print start

    # Num of Sector 
    num = struct.unpack('<L', part[12:12+4])[0] 
    size = (num * 512.) / 1024 / 1024 / 1024 
    print StringCalcSector2Size(num)
{% endhighlight %}

이제 실행 결과를 보자. 

```
MBR Read Success
PrimaryPartition
63
101.94 MB
ExtendedPartition
```

이제 주 파티션의 정보를 최종적으로 다등머 한줄에 출력되게 하였다. 

{% highlight python %}
# 주 파티션 처리
def PrimaryPartition(part) :
    start = struct.unpack('<L', part[ 8: 8+4])[0] # Starting LBA
    num   = struct.unpack('<L', part[12:12+4])[0] # Num of Sector

    print '[P] %10d  %s' % (start, StringCalcSector2Size(num))
{% endhighlight %}

실행결과가 한줄에 표시 되었다. 

```
MBR Read Success
[P]         63  101.94 MB
ExtendedPartition
```

##5. 확장 타피션 처리하기 
확장 파티션은 주 파티션에 비해 까다롭다. 확장 파티션안에는 여러개의 논리 드라이브가 올 수 있다. 따라서 확장 파티션에서는 섹터의 수는 중요하지 않고 Starting LBA의 위치가 필요한 정보이다. 

{% highlight python %}
# 확장 파티션 처리
def ExtendedPartition(part) :
    print 'ExtendedPartition'
    start = struct.unpack('<L', part[ 8: 8+4])[0] # Starting LBA
    print start
{% endhighlight %}

실행 결과를 보자. 

```
MBR Read Success
[P]         63  101.94 MB
ExtendedPartition
208845
```

위 결과에서 출력된 Starting LBA는 향후 계속적으로 논리드라브 및 다음 확장 논리드라이브를 담고 있는 파티션 테이블을 가진 위치의 Base로 사용될 중요 정보이다. 따라서 이는 별도로 보관해 둘 필요가 있다. 

{% highlight python %}
# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    print base

… (중략) …
    
if __name__ == '__main__' :
    # HDD1를 연다
    handle = open('\\\\.\\PhysicalDrive1', 'rb')

    # MBR (0번 섹터) 위치로 이동한다
    handle.seek(0 * 512) # Offset이므로 512를 곱함

    # MBR 역역을 읽는다
    mbr = handle.read(0x200)

    # MBR의 Magic ID가 맞나?
    if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA :
        print 'MBR Read Success'
        
        # 파티션 정보 획득
        part = []
        
        for i in range(4) :
            part.append(mbr[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])
        
        # 각 파티션의 시스템 타입만 출력
        for i in range(4) :
            p = part[i]
            if ord(p[4]) == 0xF or ord(p[4]) == 0x5 :
                base = struct.unpack('<L', p[ 8: 8+4])[0] # Base
                ExtendedPartition(p, 0)  # 확장 파티션
            elif ord(p[4]) != 0 :
                PrimaryPartition(p)  # 주 파티션
    else :
        print 'MBR Read Fail'

    # HDD0를 닫는다
    handle.close()
{% endhighlight %}

확장 파티션이 가리키는 위치에 가면 논리 드라이브가 담긴 별도의 파티션 테이블이 존재한다. 따라서 해당 위치로 가야 한다. 이 함수는 계속적으로 재활용하기 위해서 위치 이동 하는 위치 정보를 Base를 기준으로 Starting LBA를 더해야 함으로 다음과 같이 작성하였다. 

{% highlight python %}
# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    print base
    
    handle.seek((base+start) * 512)
    data = handle.read(0x200)
    
    # BR의 Magic ID가 맞나?
    if ord(data[510]) == 0x55 and ord(data[511]) == 0xAA :
        print 'PARTITION Read Success' 
{% endhighlight %}

해당 위치로 이동하여 512 Byte를 읽었으며, Magic ID가 존재하는지 체크한다. 아래의 결과를 통해 Magic ID가 제대로 있음을 확인할 수 있다. 

```
MBR Read Success
[P]         63  101.94 MB
ExtendedPartition
208845
PARTITION Read Success
```

읽어들인 위치에는 파티션의 정보가 2개 존재한다. 첫번째는 논리 드라이브에 대한 정보이고, 다른 하나는 다음 논리 드라이브의 정보를 담고있는 파티션 테이블이다. 따라서 2개의 정보만을 읽어들인다. 

{% highlight python %}
# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    
    handle.seek((base+start) * 512)
    data = handle.read(0x200)
    
    # BR의 Magic ID가 맞나?
    if ord(data[510]) == 0x55 and ord(data[511]) == 0xAA :
        print 'PARTITION Read Success'

        # 논리 드라이브 정보 획득
        part = []
        
        for i in range(2) :
            part.append(data[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])
{% endhighlight %}

이제 첫번째 정보가 존재한다면, 논리 드라이브 처리를 하고, 두번째 정보가 존재한다면 다시 확장 파티션 처리를 해면 된다. 

{% highlight python %}
# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    print base
    
    handle.seek((base+start) * 512)
    data = handle.read(0x200)
    
    # BR의 Magic ID가 맞나?
    if ord(data[510]) == 0x55 and ord(data[511]) == 0xAA :
        print 'PARTITION Read Success'

        # 논리 드라이브 정보 획득
        part = []
        
        for i in range(2) :
            part.append(data[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])
            
        if ord(part[0][4]) != 0x0 : # 논리 드라이브 정보가 존재하는가?
            print 'LogicalDrivePartition' # 논리 드라이브 처리
        if ord(part[1][4]) != 0x0 :
            start = struct.unpack('<L', part[1][8:8+4])[0]
            ExtendedPartition(part[1], start) # 확장 논리 드라이브 처리
{% endhighlight %}

확장 파티션의 항상 Base를 기준으로 접근해야 하기 때문에 Starting LBA를 인자하여 계속 확장 파티션을 처리하게끔 자기 자신을 호출하게 된다. 실행결과는 아래와 같다. 

```
MBR Read Success
[P]         63  101.94 MB
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
```

##6. 논리 드라이브 처리하기 
마지막으로 논리 드라이브를 처리해보자. 논리 드라이브 처리도 편하게 하기 위해 함수로 분리하였다. 

{% highlight python %}
# 논리 드라이브 처리
def LogicalDrivePartition(part, base) :
    print 'LogicalDrivePartition'

# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    
    handle.seek((base+start) * 512)
    data = handle.read(0x200)
    
    # BR의 Magic ID가 맞나?
    if ord(data[510]) == 0x55 and ord(data[511]) == 0xAA :
        print 'PARTITION Read Success'

        # 논리 드라이브 정보 획득
        part = []
        
        for i in range(2) :
            part.append(data[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10]) 

        if ord(part[0][4]) != 0x0 : # 논리 드라이브 정보가 존재하는가?
            LogicalDrivePartition(part[0], base+start)  # 논리 드라이브 처리
        if ord(part[1][4]) != 0x0 :
            start = struct.unpack('<L', part[1][8:8+4])[0]
            ExtendedPartition(part[1], start) # 확장 논리 드라이브 처리
{% endhighlight %}

논리 드라이브도 주 파티션과 같이 위치와 용량을 출력해보자. 

{% highlight python %}
# 논리 드라이브 처리
def LogicalDrivePartition(part, base) :
    print 'LogicalDrivePartition'
    start = struct.unpack('<L', part[ 8: 8+4])[0] # Starting LBA
    num   = struct.unpack('<L', part[12:12+4])[0] # Num of Sector
    
    print '[L] %10d  %s' % (base+start, StringCalcSector2Size(num))
{% endhighlight %}

소스코드에서 볼 수 있듯이 논리 드라이브는 새로운 Base 위치에서 떨어진 상대 거리의 Starting LBA를 가진다. 따라서 시작위치에서는 두 값을 더해서 그 위치를 출력하고 있다. 실행 결과는 다음과 같다. 

```
MBR Read Success
[P]         63  101.94 MB
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
[L]     208908  196.08 MB
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
[L]     610533  400.03 MB
ExtendedPartition
PARTITION Read Success
LogicalDrivePartition
[L]    1429848  321.58 MB
```

실행결과에서 필요한 출력 값만을 남기고 아래의 결과를 확인할 수 있다. 

```
[P]         63  101.94 MB
[L]     208908  196.08 MB
[L]     610533  400.03 MB
[L]    1429848  321.58 MB
```

이제 이 내용을 WinHex에서 HDD1을 읽은 결과와 직접 비교해보자. 

![](/images/2014/winhex.PNG)

용량이 소수점으로 표시된 것만을 제외하면 위치값들이 정확하게 출력되었음을 알 수 있을 것이다. 

##7. 전체 소스코드 
지금까지의 전체 소스코드는 아래와 같다. 

{% highlight python %}
# -*- coding:utf-8 -*-

import struct

# 섹터를 용량으로 변환하여 문자열로 리턴한다
def StringCalcSector2Size(num) :
    str_size = ['B', 'KB', 'MB', 'GB']
    
    t = size = (num * 512.)
    
    for i in range(4) :
        t = t / 1024    # 한번 1024를 나누어
        if int(t) > 0 : # 정수 값이 있는지를 체크 
            size = size / 1024
        else :
            break   # 정수 값이 없다면 종료 
    
    return '%.2f %s' % (size, str_size[i])  

# 논리 드라이브 처리
def LogicalDrivePartition(part, base) :
    print 'LogicalDrivePartition'
    start = struct.unpack('<L', part[ 8: 8+4])[0] # Starting LBA
    num   = struct.unpack('<L', part[12:12+4])[0] # Num of Sector
    
    print '[L] %10d  %s' % (base+start, StringCalcSector2Size(num))
    
    
# 확장 파티션 처리
def ExtendedPartition(part, start) :
    print 'ExtendedPartition'
    
    handle.seek((base+start) * 512)
    data = handle.read(0x200)
    
    # BR의 Magic ID가 맞나?
    if ord(data[510]) == 0x55 and ord(data[511]) == 0xAA :
        print 'PARTITION Read Success'

        # 논리 드라이브 정보 획득
        part = []
        
        for i in range(2) :
            part.append(data[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10]) 

        if ord(part[0][4]) != 0x0 : # 논리 드라이브 정보가 존재하는가?
            LogicalDrivePartition(part[0], base+start)  # 논리 드라이브 처리
        if ord(part[1][4]) != 0x0 :
            start = struct.unpack('<L', part[1][8:8+4])[0]
            ExtendedPartition(part[1], start) # 확장 논리 드라이브 처리


# 주 파티션 처리
def PrimaryPartition(part) :
    start = struct.unpack('<L', part[ 8: 8+4])[0] # Starting LBA
    num   = struct.unpack('<L', part[12:12+4])[0] # Num of Sector

    print '[P] %10d  %s' % (start, StringCalcSector2Size(num))

if __name__ == '__main__' :
    # HDD1를 연다
    handle = open('\\\\.\\PhysicalDrive1', 'rb')

    # MBR (0번 섹터) 위치로 이동한다
    handle.seek(0 * 512) # Offset이므로 512를 곱함

    # MBR 역역을 읽는다
    mbr = handle.read(0x200)

    # MBR의 Magic ID가 맞나?
    if ord(mbr[510]) == 0x55 and ord(mbr[511]) == 0xAA :
        print 'MBR Read Success'
        
        # 파티션 정보 획득
        part = []
        
        for i in range(4) :
            part.append(mbr[0x1BE + (i*0x10):0x1BE + (i*0x10) + 0x10])
        
        # 각 파티션의 시스템 타입만 출력
        for i in range(4) :
            p = part[i]
            if ord(p[4]) == 0xF or ord(p[4]) == 0x5 :
                base = struct.unpack('<L', p[ 8: 8+4])[0] # Base
                ExtendedPartition(p, 0)  # 확장 파티션
            elif ord(p[4]) != 0 :
                PrimaryPartition(p)  # 주 파티션
    else :
        print 'MBR Read Fail'

    # HDD0를 닫는다
    handle.close()
{% endhighlight %}

##8. 결론 
파이썬을 이용하면 자신에게 맞는 포렌식 도구를 쉽게 개발 할 수 있다. 이는 파이썬 언어의 간결함과 컴파일과 같은 복잡한 과정이 필요 없으며 다양한 운영체제를 지원하기 때문에 가능한 일이다. 

파일시스템의 원리를 정확히 인지하고 있다면 지금은 파티션 정보를 확인하는 소스코드를 만들었지만, 이는 계속적으로 확장하여 삭제된 파일등을 복구하는 툴로도 발전할 수 있을 것이다.




***



####Update



- 2015-08-26 : 중복 소스코드 제거
- 2013-04-14 : 최초로 작성