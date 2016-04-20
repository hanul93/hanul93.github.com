---
layout: post
title: 파이썬 소스코드 문서화하기
description: 프로그램을 개발하다보면 문서화 작업이 소홀할 경우 향후 유지보수시 어떤 의도로 프로그램이 개발되었는지 파악하기가 힘들때가 있다. 파이썬에는 스핑크스(Sphinx)라는 문서화 도구가 존재한다. 이를 이용해서 어떻게 문서화를 하는지 알아보기로 하자.
date: 2016-04-21 06:58:00 
tags: 파이썬 스핑크스 문서화 도구 Sphinx
---
  
## 1. 들어가며

프로그램 개발이 끝나고 서비스가 진행된다. 프로그램의 장애 혹은 새로운 요구사항이 발생해서 프로그램의 소스코드를 변경해야 한다. 일명 유지보수라 불리는 이 시점에 원 개발자가 아닌 다른 개발자가 프로그램 소스를 인계 받아서 작업하는 경우 이전 개발자가 어떤 의도를 가지고 코딩을 했는지 불분명한 경우가 있다. 프로그램에 대한 문서를 들여다보지만 아무런 내용이 없다(?).

프로그램을 개발하다보면 문서화 작업이 소홀할 때가 있다. 나중에 문서화해야지 하고 마음 먹어보지만 그것도 쉽지 않은 일이다. 결국 시간이 좀 걸리긴 하나 프로그램 개발과 문서화 작업은 병행해야 한다.

요즘처럼 순간적인 아이디어를 구현해보고 사용자들의 반응을 살펴본 뒤 본격적으로 런칭하는 방법이 주류를 이룬다. 즉 빠른 설계 및 구현을 목표로 한다. 이때문에 개발자들은 문서화할 시간이 더더욱 부족하다고 말할지도 모른다. 하지만 파이썬 개발자들에게는 개발하면서 바로 문서화까지 작업할 수 있는 스핑크스(Sphinx)라는 좋은 도구가 있다. 


## 2. 스핑크스(Sphinx) 설치 및 실행

스핑크스는 파이썬 홈페이지의 온라인 메뉴얼을 만든 도구라면 이해가 될 것이다. 

* <https://docs.python.org/2/index.html>

이제 여러분들이 만든 프로그램도 스핑크스만 사용하면 파이썬의 온라인 메뉴얼과 같은 멋진 메뉴얼을 만들 수 있다.
 

### 설치하기

스핑크스는 간단하게 ```pip```를 이용해서 쉽게 설치할 수 있다. 

```
pip install Sphinx
```

설치하는 동안 사용자들의 환경이 워낙 다양하다보니 이런 저런 오류를 만날 수 있을 것이다. [^1]

<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong>오류 발생 </strong></span></legend><!--Creative Commons License--><div style="float: left; width: 88px; margin-top: 3px;"><img alt="Creative Commons License" style="border-width: 0" src="/images/exclamationmark.png"/></div><div style="margin-left: 92px; margin-top: 3px; text-align: justify;">

설치시 오류가 발생한다면 <code class="highlighter-rouge">pip</code> 버전이 낮아서 발생할 가능성이 크다. 따라서 <code class="highlighter-rouge">pip</code>를 업그레이드해서 다시 인스톨하면 된다.

<div class="highlighter-rouge"><pre class="highlight"><code>easy_install pip    
pip install --upgrade setuptools
</code></pre>
</div>
 
</div></fieldset>

### 프로젝트 문서화 환경 설정

문서화할 프로젝트가 존재하는 폴더에 위치한 뒤 sphinx-quickstart 실행한다. 

```
C:\Python27\Scripts\sphinx-quickstart.exe
```

스핑크스를 실행하면 추가적으로 필요한 모듈들도 존재한다. 따라서 실행시 오류가 발생한다면 오류 메시지를 잘 확인해서 필요한 모듈도 설치하기 바란다. [^2]

<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong>오류 발생 </strong></span></legend><!--Creative Commons License--><div style="float: left; width: 88px; margin-top: 3px;"><img alt="Creative Commons License" style="border-width: 0" src="/images/exclamationmark.png"/></div><div style="margin-left: 92px; margin-top: 3px; text-align: justify;">

만약 실행시 아래와 같은 오류가 나온다면 <code class="highlighter-rouge">easy_install markupsafe</code>을 실행하여 추가 설치한다.

<div class="highlighter-rouge"><pre class="highlight"><code>ImportError: No module named markupsafe    
</code></pre>
</div>
 
</div></fieldset>

sphinx-quickstart가 실행되면 아래와 같이 화면을 볼 수 있는데 몇가지 질문들에 대한 답을 하면 되는데 아래 그림처럼 셋팅하자.

![](/images/2016/sphinx/start.png) 

### 문서화 자동 생성 도구 실행

sphinx-apidoc 프로그램을 이용해서 자동으로 문서 생성에 필요한 파일들을 생성한다. 도움말을 보고 싶다면 ```--help```를 입력하면 된다. 아래의 그림은 ```-F -o 결과 폴더] [문서화 대상 소스 폴더] [문서화 대상 제외 폴더...]```를 차레로 입력한 것이다.

![](/images/2016/sphinx/apidoc.png) 

### HTML 파일 생성

문서화 대상 소스의 위치에 따라 ```conf.py```을 수정해야 한다. ```conf.py```파일을 열어 소스코드 상단 부분에 ```sys.path.insert``` 함수를 이용해서 문서화 대사 소스의 위치를 추가한다.

```
sys.path.insert(0, os.path.abspath('.'))
sys.path.insert(0, os.path.abspath('..'))
```

이후에 ```docs``` 폴더에 존재하는 ```make.bat```를 아래와 같이 실행하면 문서화 대상 소스코드를 분석해서 HTML 파일을 생성하게 된다.

![](/images/2016/sphinx/html.png) 


오류 발생한다면 아래의 패키지를 추가 설치한다.

```
pip install alabaster # 테마 패키지
pip install pytz
```

테마를 새롭게 설치하였으므로 ```docs/conf.py``` 파일에서 ```html_theme_path``` 내용을 찾아 아래와 같이 수정한다. [^3]

```
html_theme_path = [r'C:\Python27\Lib\site-packages']
```

프로그램이 정상적으로 수행되었다면 ```docs/_build/html/index.html``` 파일이 생성된다. 이 파일을 웹브라우저로 열면 아래와 같은 화면을 볼 수 있다.

![](/images/2016/sphinx/index.png) 

### 테마 변경

스핑크스는 다양한 테마를 지원하고 있으며 쉽게 테마를 변경할 수도 있다. ```conf.py``` 내용중 ```html_theme```를 아래와 같이 수정한 뒤 다시 ```make.bat html```을 실행하면 된다.

```
html_theme = 'classic'
```

스핑크스가 지원하는 테마들은 다음과 같다. [^4]

![](/images/2016/sphinx/theme1.png) 
![](/images/2016/sphinx/theme2.png) 

### 다이어그램 생성

문서화를 진행하다보면 클래스의 상속관계등에 대한 다이어그램을 보고 싶을 때가 있다. 스핑크스에서도 다디어그램을 생성해서 보여주는 기능이 내장되어 있다. 

먼저 ```conf.py```에 확장 모듈 내용을 추가한다. 더 많은 확장 모듈은 아래의 URL에서 확인할 수 있다.

* <http://www.sphinx-doc.org/en/stable/extensions.html>

```
extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.intersphinx',    
    'sphinx.ext.todo',
    'sphinx.ext.viewcode',
    'sphinx.ext.graphviz',           # 추가
    'sphinx.ext.inheritance_diagram' # 추가
]
```

이제 다이어그램을 보기 원하는 소스코드의 ```.rst``` 파일을 열어 다음의 내용을 추가한다.

```
hwp3x module
============

.. automodule:: hwp3x
    :members:
    :undoc-members:
    :show-inheritance:
.. inheritance-diagram:: hwp3x  # 추가
   :parts: 1                    # 추가
```   

다시 ```make.bat html``` 명령어를 실행하면 웹페이지에 특정 모듈 설명 페이지 하단에 다이어그램이 그려진다. 아래는 간단한 스핑크스에 의해 생성된 다이어그램이다.

![](/images/2016/sphinx/inheritance-c390470570dc118f9aa55caac480e9ea4918828a.png) 

```.rst``` 파일에 추가한 ```:parts:```의 값을 ```2```로 수정하면 다이어그램은 아래와 같이 바뀐다.

![](/images/2016/sphinx/inheritance-b6150ae2e1a54ee1ebc656a20f9a8c2b8cc40df6.png) 




## 3. 스핑크스(Sphinx) 문법

앞에서 우리는 스핑크스에 대한 설치와 간단한 실행방법에 대해서는 알아보았다. 이제 소스코드의 문서화 내용이 스핑크스를 통해 웹페이지에 표현될 수 있도록 스핑크스 문법에 대해서 살펴보도록 하겠다. (문법에 대한 부분은 참고 링크를 확인하기 바란다. [^5][^6])

### 모듈 설명

모듈 설명은 소스코드 제일 상단에 모듈의 전체적인 이해를 위해 작성하는 문서이다.

```
"""
====================================
 :mod:`xscanengine` 모듈 
====================================
.. moduleauthor:: 최원혁 <hanul93@gmail.com>

설명
=====

모든 Scan 엔진을 위한 Base 클래스 및 상수가 정의되어 있다.

참고
====

관련 링크:
 * https://docs.python.org/2/library/abc.html
 * http://stackoverflow.com/questions/4714136/python-how-to-implement-
 virtual-methods

관련 작업자
===========

 * 최원혁 (Kei Choi)

작업일지
--------

 * 2016.04.19 Kei : 검사 엔진이 구현해야 할 함수 정의

 """
```

위의 결과물이 아래의 그림이다.

![](/images/2016/sphinx/screen1.png) 


### 클래스 설명

문서화 될 내용은 항상 해당 클래스, 함수, 전역변수 선언 아래 줄부터 작성한다.

```
class XScanEngine (object) :
    '''XScanEngine 클래스
    
    모든 검사 엔진들은 XScanEngine 클래스를 반드시 상속 받아야 한다.
    
    예제 :
        
        >>> class HWP3(XScanEngine) :
        ...
    '''
```

위의 결과물이 아래의 그림이다.

![](/images/2016/sphinx/screen2.png) 


### 함수 설명

함수를 설명할 때에는 인자값, 리턴값등을 추가한다. 이때 사용되는 키워드는 ``` :param:```, ```:returns:```이다.

```
    @abstractmethod
    def GetPPSTree(self) :
        '''분석 대상 파일의 내부 Tree 구조를 리턴하는 가상함수이다.
        
        Tree 구조는 다음과 같다.
        
        * ['1', '2', '3'] : 1, 2, 3 모두 스트림임
        * [['1', ['2']], '3'] : 1은 소토리지이며 1 하위에 2 스트림이 존재함
        
        :returns list: tree - 파일 내부 Tree 구조를 표현한 리스트
        '''
        raise NotImplementedError
```        

위의 결과물이 아래의 그림이다.

![](/images/2016/sphinx/screen3.png) 

## 4. 결론

문서화 도구는 프로그램 개발시 간혹 묵인되기도 한다. 대부분 개발자들끼리 소스코드 리뷰등을 통해 소스코드를 하나하나 함께 검토를 해가지만 **"문서화 리뷰"**라는 말은 못들어보지 않았는가? 

하지만 제품 개발이 완료되고 유지보수할때 문서화 된 자료가 아무것도 없다면 이는 유지보수에 큰 어려움이 다를 수 밖에 없다. 

지금이라도 늦지 않았다. 현재 진행하고 있는 프로젝트의 가장 많이 사용하는 클래스, 함수부터라도 조금씩 문서화를 스핑크스를 통해 진행해보자.

***

#### Update

- 2016-04-21 : 최초로 작성

#### 참고 링크

[^1]: [easy_install.exe Permission Denied on Windows 8 ](http://stackoverflow.com/questions/17601020/easy-install-exe-permission-denied-on-windows-8)
[^2]: [ImportError: No module named markup safe #6](https://github.com/rdickert/project-quicksilver/issues/6)
[^3]: [Failed to build documentation #24](https://github.com/neurodroid/stimfit/issues/24)
[^4]: HTML theming support : <http://www.sphinx-doc.org/en/stable/theming.html>
[^5]: 스핑크스(sphinx)를 이용한 파이썬 API 문서화 : <http://egloos.zum.com/mcchae/v/11080328>
[^6]: Python 문서화, Sphinx로 아주 간단하게 시작해보기 : <http://b.ssut.me/65>