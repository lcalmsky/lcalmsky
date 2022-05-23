`GitHub markdown`에서 수학 표현식 렌더링을 제공합니다. (2022년 5월 19일부터!)

[`LaTeX` 문법](https://ko.wikipedia.org/wiki/%EC%9C%84%ED%82%A4%EB%B0%B1%EA%B3%BC:TeX_%EB%AC%B8%EB%B2%95)으로 `inline` 또는 `code block` 형태로 작성할 수 있습니다.

## block으로 사용

주변 텍스트와 별개로 수학 표현식을 추가하기 위해 사용합니다.

`$$`로 새로운 라인을 시작하면 수학 표현식으로 인식합니다.

아래 처럼 입력했을 경우,

```text
**The Cauchy-Schwarz Inequality**
$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$
```

![](https://i0.wp.com/user-images.githubusercontent.com/7219923/165633956-cfe6be6d-66ec-451a-8afd-be3546f4d0a1.png?ssl=1)

이렇게 표시됩니다.

## inline으로 사용

텍스트 중간에 수학식을 입력해야 할 경우 `$`를 사용합니다.

아래 처럼 입력했을 경우,

```text
This sentence uses `$` delimiters to show math inline:  $\sqrt{3x-1}+(1+x)^2$
```

![](https://i0.wp.com/user-images.githubusercontent.com/7219923/165633994-a1f2cee5-71cb-4303-bbd9-1df1604d7a01.png?ssl=1)

이렇게 표시됩니다.

> 현재 `GitHub markdown`에서만 지원하는 기능이므로 블로그에서 제공하는 `markdown`으로는 수학표현식을 사용할 수 없습니다.
> 
> 알고리즘을 풀 때 수학표현식을 작성해야 할 경우가 많이 있는데, 그 때 사용하면 유용할 거 같습니다.

## 렌더링 방식

`GitHub`의 수학 렌더링 기능은 `JavaScript`기반 디스플레이 엔진 오픈소스인 `MathJax`를 사용합니다. `MathJax`는 광범위한 `LaTeX` 매크로와 유용한 접근성 확장을 지원합니다.

렌더링 된 결과는 `CHTML` 또는 `SVG` 형식을 지원합니다. 즉, 위의 수식을 `LaTex` 문법으로 작성했을 경우 `HTML` 또는 `SVG` 형태로 변경됩니다.

수식에서 우클릭하면 아래와 같은 메뉴들을 확인할 수 있습니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/docs/blog/etc/example.png)