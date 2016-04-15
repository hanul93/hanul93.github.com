---
layout: post
title: PyV8을 이용한 자바스크립트 악성코드 분석하기
description: 자바스크립트를 이용한 악성코드는 난독화등으로 인해 분석하기도 힘들고 진단용 백신을 만들기도 어렵다. 여기에서는 간단하게 PyV8을 이용한 방법을 소개한다.
date: 2016-04-09 08:10:00 
tags: 파이썬 PyV8 자바스크립트 악성코드 분석
---
  
## 1. 들어가며

악성코드를 분석하다보면 자바스크립트와 같은 텍스트 기반의 악성코드들은 백신을 만들기에 너무 까다로운 존재이다. 특히 난독화된 자바스크립트의 경우 분석도 쉽지 않고 이를 진단하는 백신을 만들기도 어렵기만하다. 

유사한 형태로 다형성 바이러스가 존재한다. 매 감염때마다 형태가 바뀌고 분석하기도 까다롭고 백신의 시그너처를 선정하기도 힘들기만 했다. 결국 다형성 바이러스를 진단하기 위해 에뮬레이터를 백신에 탑재하는 방법을 선택하게 되었다.

자바스크립트 역시 에뮬레이터를 이용하는 방법이라면 좀더 수월하게 분석하고 백신을 만들수가 있다. 여기에서는 PyV8[^1]을 이용한 방법을 소개하고자 한다. 

## 2. PyV8

PyV8은 구글에서 개발한 자바스크립트 엔진인 V8의 파이썬 래퍼이다. PyV8은 자바스크립트 소스를 해석하고 실행하여 그 결과를 파이썬으로 전달한다. PyV8은 아래의 URL에서 다운로드 받아 설치할 수 있다.

* <https://code.google.com/archive/p/pyv8/downloads>

설치가 완료되었다면 간단히 다음과 같이 PyV8을 사용할 수 있다.

```
>>> import PyV8
>>> ctx = PyV8.JSContext() # PyV8 사용 준비
>>> ctx.enter()
>>> ctx.eval('var i = 1+1') # 자바스크립트 실행
```

실행한 결과를 보는 방법은 두가지가 있다.

### (1) eval 함수를 사용하는 방법

앞에서 실행한 방법처럼 ```eval``` 함수를 실행하여 처리하는 방식이다.

```
>>> ret = ctx.eval('i') # 자바스크립트 변수를 직접 입력
>>> ret
2
```

### (2) locals 클래스를 사용하는 방법

PyV8은 자바스크립트를 실행하면서 생성된 모든 변수를 ```locals``` 클래스에 보관을 하고 있다. 

```
>>> ctx.locals.keys() # 자바스크립트의 전체 변수명을 획득 
['i']
>>> ret = ctx.locals['i'] # i 변수에 저장된 값을 확인
>>> ret
2
```


## 3. Locky 랜섬웨어 분석 (자바스크립트 Part)

### (1) Locky 랜섬웨어

Locky 랜섬웨어는 E-mail의 첨부파일로 확산이 된다. 첨부파일은  2개의 자바스크립트이 zip으로 압축된 형태이며 이중 하나의 자바스크립트가 아래 그림과 같다.

![](/images/2016/pyv8/locky.png) 

소스가 난독화 되어 있어 쉽게 분석하기는 어려운 형태이다. 이를 PyV8을 이용하여 실행하면서 분석해 본다.

### (2) Locky 랜섬웨어 에뮬레이션

Locky 랜섬웨어 자바스크립트 소스코드를 로딩한 후 PyV8을 이용하여 실행한다. 

```
>>> import PyV8
>>> s = open('locky2.js_', 'rb').read() # 랜섬웨어 소스코드 로딩
>>> ctx = PyV8.JSContext()
>>> ctx.enter()
>>> ctx.eval(s) # 랜섬웨어 실행
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ReferenceError: ReferenceError: WScript is not defined (  @ 167 : 0 )
-> WScript.Sleep(10);
>>> _
```

PyV8이 실행되면서 ```WScript```가 선언되어 있지 않다며 실행을 종료한다. 사실 PyV8은 자바스크립트 문법을 해석해주는 역활을 할 뿐 실제 다양한 운영체제의 API를 준비하고 있지 않다. 그렇기 때문에 우리는 PyV8을 이용하여 악성코드를 실행하는 가감한 행동을 할 수 있다. 어차피 PyV8은 파일을 삭제하거나 생성하고 실행하는 행동은 하지 못한다. 운영체제의 API가 없기 때문에... 

### (3) PyV8과 Dummy 운영체제 API 연결

PyV8이 악성코드를 잘(?) 실행할 수 있도록 Dummy 운영체제 API를 생성하여 연결하는 작업을 진행해 보자. 아래의 소스코드는 Locky 랜섬웨어가 필요로 하는 운영체제 API를 Dummy로 생성하였다.

```
class K2WScript(PyV8.JSClass) :
    def Sleep(self, x) :
        time.sleep(x/1000.)
        
    def CreateObject(self, progid) :
        print '[*] CreateObject :', progid
        if progid.lower() == 'wscript.shell' :
            return K2WshShell()
        elif progid.lower() == 'msxml2.xmlhttp' :
            return K2XMLHTTP()
        elif progid.lower() == 'adodb.stream' :
            return K2Stream()
            
class K2WshShell(PyV8.JSClass) : 
    def ExpandEnvironmentStrings(self, x) :
        s = _winreg.ExpandEnvironmentStrings(unicode(x))
        print '[*] ExpandEnvironmentStrings :', x
        print '    [-] :', s
        return s
    def Run(self, x, y, z) :
        print '[*] WshShell.Run :'
        print '    [-] :', x  
        
class K2XMLHTTP(PyV8.JSClass) : 
    def open(self, x, y, z) :
        print '[*] XMLHTTP.open :'
        print '    [-] :', x  
        print '    [-] :', y  
        print '    [-] :', z
    def send(self) :
        pass
        
class K2Stream(PyV8.JSClass) :
    def open(self) :
        pass
    def write(self, x) :
        pass
    def SaveToFile(self, x, y) :
        print '[*] SaveToFile :', x
    def close(self) :
        pass
        
class Global(PyV8.JSClass):
    WScript = K2WScript()
```

이제 PyV8을 초기화 할 때 사용했던 ```JSContext``` 함수의 인자값으로 ```Global``` 클래스를 입력한다.

```
>>> ctx = PyV8.JSContext(Global())
```

나머지 부분은 사용방법이 동일하다.

### (4) Locky 랜섬웨어 에뮬레이션 결과

아래는 PyV8에 Dummy 운영체제 API를 연결하여 Locky 랜섬웨어를 실행한 결과이다.

![](/images/2016/pyv8/locky_run.png) 

결과를 보면 특정 웹사이트에서 파일을 다운로드하여 TEMP 폴더에 저장한 후 실행하는 것을 확인할 수 있다.


## 4. 결론

PyV8을 이용하면 자바스크립트로 작성된 악성코드를 쉽게 분석할 수 있다. 또한 자바스크립트를 실행하지만 악성코드에 감염되지 않으므로 시스템이 안전하다. 하지만 PyV8은 운영체제의 API를 지원해주지 않으므로 이 부분은 직접 코딩을 통해 Dummy 운영체제 API를 작성해야 하는 불편함이 있다. 어차피 자바스크립트의 악성코드들이 사용하는 운영체제 API는 한정적이다. 

한번만 수고스럽더라도 Dummy 운영체제 API를 만들어 계속 보강해 간다면 조만간 나만의 강력한 자바스크립트 악성코드 분석환경을 가지게 될것이다.

***

#### Update

- 2016-04-09 : 최초로 작성


[^1]: PyV8 : <https://pypi.python.org/pypi/PyV8>
