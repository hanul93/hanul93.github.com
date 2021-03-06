---
layout: post
title: 키콤백신 개발 이야기 - 코딩 스타일 가이드라인
description:  키콤백신을 개발하면서 고민한 내용들을 적은 이야기이다. 여기에서는 코딩 스타일 가이드라인에 대해서 이야기한다.
date: 2015-09-03 09:00:00 
tags: 파이썬 키콤백신 PEP8
---
  
  
<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong>읽기 전에 </strong></span></legend><!--Creative Commons License--><div style="float: left; width: 88px; margin-top: 3px;"><img alt="Creative Commons License" style="border-width: 0" src="/images/exclamationmark.png"/></div><div style="margin-left: 92px; margin-top: 3px; text-align: justify;">본 내용은 키콤백신을 개발하면서 고민했던 내용들을 적은 것입니다. 따라서 다분히 개인적인 의견들이므로 보시는 분들에 따라서 이견을 가지실 수 있습니다. 그냥 "이 사람은 이렇게 개발하는구나" 하고 봐 주시면 좋겠습니다. 
</div></fieldset>


## 들어가며
오픈소스로 프로젝트를 진행하게 되면 많은 사람들이 프로젝트에 참여하게 될 것이다. 그 만큼 소스코드에 대한 수정이 다양하게 이루어지게 되며 많은 프로그래머들이 각자의 코딩 스타일로 소스 코드를 추가하고 수정 하게 된다. 

다양한 코딩 스타일로 작성된 소스 코드는 수정이 필요할 경우 소스 코드를 해석하는데 상당히 많은 시간을 소비하게 되는 문제를 야기하게 된다. 결국 좋은 품질의 소스 코드를 확보하기 위해서는 각자가 가진 코딩 스타일을 통일시키는 것이 오픈소스로 프로젝트를 진행하는데 도움이 된다. 

## 코딩 스타일 가이드라인이란?

코딩 스타일 가이드라인(또는 코딩 컨벤션이라고도 함)은 프로그램 코드를 작성할 때 사용되는 프로그래머들 사이의 약속이다. 예를 들면 들여쓰기는 탭이로 할 것인지 아니면 공백으로 할 것인지부터 변수명은 어떤 방법으로 선언할 것인지를 결정하는 것이 대표적인 코딩 스타일 가이드라인이다.

프로그래밍 언어별로 나름 표준화 된 코딩 스타일 가이드라인이 존재하는데 키콤백신 프로젝트[^1]는 파이썬으로 작성되었으므로 파이썬의 표준 코딩 스타일 가이드라인인 PEP8[^2]을 따른다.

## PEP (Python Enhance Proposal)

PEP란 파이썬을 개선하기 위한 개선 제안서를 뜻하는 것으로 3가지로 구분된다.

* Standard Track : 파이썬의 새로운 기능이나 구현을 제안
* Informational : 파이썬의 디자인 이슈나 일반적인 지침을 제안
* Process : 파이썬 커뮤니티에 정보를 제안

이들 PEP 제안 중에서 PEP8은 파이썬 코딩 스타일 가이드라인을 의미하며 파이썬 커뮤니티에 정보를 제안하는 Process에 해당된다.

## PEP8 : 파이썬 코딩 스타일 가이드라인

간단히 PEP8의 내용은 다음과 같다.

1. 들여쓰기는 공백 4개로 한다.
2. 한 줄의 최대 글자는 79자로 한다.
3. 파일은 UTF-8 또는 ASCII로 인코딩한다.
4. 하나의 import에는 모듈 하나만 한다.
5. import는 표준 라이브러리, 서드파티, 로컬 라이브러리 순서로 묶는다.
5. 소괄호, 중괄호, 대괄호 사이에 추가로 공백을 입력하지 않는다.

자세한 PEP8의 내용은 <https://www.python.org/dev/peps/pep-0008> 에서 확인할 수 있다.

## PyCharm을 이용한 PEP8  체크하기

키콤백신은 PyCharm을 이용해서 개발하고 있는데 PyCharm에는 PEP8인 파이썬 코딩 스타일 가이드라인을 체크하는 기능이 기본적으로 활성화 되어 있다.

혹시라도 PEP8이 체크되어 있지 않다면 PyCharm 메뉴에서 ```File → Settings```를 선택한 후 아래의 Setting 윈도우에서 PEP8 항목을 찾아서 체크하면 된다.

![](/images/2015/kicomav/pep8_1.png)

아래 그림은 PyCharm에서 PEP8 옵션을 체크했을 때와 하지 않았을 때를 비교한 것이다.

![](/images/2015/kicomav/pep8_2.png)

좌측은 PEP8 옵션이 체크되지 않은 반면 우측은 PEP8 옵션이 체크되어 PEP8에 위배되는 영역은 소스 코드에 붉은색 물결 표시로 밑줄이 나타난다.

## PEP8 자동으로 체크하기

PyCharm을 이용하면 위에서 보았듯이 PEP8 파이썬 코딩 스타일 가이드라인에 위배되는 소스 코드에는 밑줄로 표시가 되지만 프로그래머가 코딩을 하면서 이 표시 여부에 집중하는 것은 무척이나 귀찮은 일이다. 따라서 키콤백신은 최종 빌드 과정에서 모든 소스 코드에 대해 PEP8을 자동으로 체크하게 된다. 이는 파이썬 외부 패키지인 pep8 1.6.2[^3]를 사용하여 처리하게 된다.

## PEP8을 따라야 하나?

파이썬 프로그래머 제작자들 사이에서도 PEP8을 따라야하는지에 대한 논쟁은 계속 진행 중이다. PEP8의 본문에도 다음과 같은  글[^4]이 있다.

> A Foolish Consistency is the Hobgoblin of Little Minds. 
> (멍청하게일관성을 유지하는 것은 소인배의 발상이다.)

세상살이도 별반 다르지 않다고 생각한다. 세상에는 법과 규칙이 존재하며 이를 어겨서는 안되지만 어떤 상황에서는  예외사항이 존재하기도 한다. 역시 PEP8을 최대한 지키되 상황에따라 예외를 줄 수도 있다. 그 예외사항을 2가지로 볼 수 있는데 아래와 같다.

**1. 가독성을 해치는 경우**

<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong></strong></span></legend><!--Creative Commons License--><div style="margin-left: 3px; margin-top: 3px; text-align: justify;">경우에 따라 기존 코드보다 PEP8이 적용된 코드가 가독성이 떨어질 수 있다. PEP8을 적용하는 이유가 가독성과 무관하지 않으므로 가독성이 좋은쪽을 따르는게 맞다고 보는것이다.
</div></fieldset>

**2. PEP8이 적용되지 않은 기존 코드를 PEP8을 적용하는 과정인 경우**

<fieldset style="margin:20px 0px 20px 0px;padding:5px;"><legend><span><strong></strong></span></legend><!--Creative Commons License--><div style="margin-left: 3px; margin-top: 3px; text-align: justify;">기존 코드의 양이 많다보면 한번에 PEP8이 적용되지 않을 수 있다. 이 경우 리펙토링등의 단계를 거쳐 순차적으로 정리해가는 과정일 가능성이 크므로 당장의 PEP8 규칙에 위배되는 코드들이 존재할 수 있다.
</div></fieldset>

우선 키콤백신 프로젝트는 기존 코드가 존재하지 않으며(물론 C/C++ 코드는 존재한다) 파이썬으로 처음부터 작성하기 때문에 당장 2번의 예외 상황이 적용되지는 않는다. 즉, PEP8을 적용하였을 때 가독성이 떨어진다고 판단 될 경우에는 예외 사항을 적용할 것이다.


[^1]: 키콤백신 프로젝트 : <http://www.kicomav.com>
[^2]: PEP8 : <https://www.python.org/dev/peps/pep-0008>
[^3]: pep8 1.6.2 : <https://pypi.python.org/pypi/pep8/1.6.2> 
[^4]: 관련 링크 : [A Foolish Consistency is the Hobgoblin of Little Minds. ](https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds)

***

#### Update

- 2015-09-03 : 최초로 작성


