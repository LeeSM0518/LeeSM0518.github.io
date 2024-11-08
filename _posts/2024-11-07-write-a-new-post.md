---
title: Writing a New Post
date: 2024-11-07 18:00:00 +0900
categories: blog
tags:
  - blog
description: jekyll-theme-chirpy 를 활용한 Github Blog 게시글 작성 가이드
---
## Naming And Path
`_posts` 위치에 `YYYY-MM-DD-TITLE.md` 파일을 생성한다.

## Front Matter
다음과 같이 게시글 맨 위에 Front Matter를 작성해야 한다.
```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```

### Timezone of Date
날짜를 정확하게 기록하려면 `_config.yml` 의 시간대를 설정할 뿐만 아니라 Front Matter에 시간대를 작성해야 한다. 
Format: `YYYY-MM-DD HH:MM:SS +/-TTTT` , e.g. `+0800`

### Categories and Tags
각 게시물의 `categories` 는 최대 두 개의 요소를 포함할 수 있으며, `tags` 는 개수 제한이 없다.
```yaml
---
categories: [Animal, Insect]
tags: [bee]
---
```

### Author Information
게시물의 작성자 정보는 설정 파일(`_data/auhors.yml`) 의 `social.name` 과 `social.links` 에서 가져온다.
```yaml
<author_id>:
  name: <full name>
  twitter: <twitter_of_author>
  url: <homepage_of_author>
```

### Post Description
게시물의 첫 단어는 게시물 목록의 홈페이지, RSS 피드의 XML에 표시되는 데 사용된다.
다음과 같이 Front Matter의 설명 필드를 사용하여 정의할 수 있다.
```yaml
---
description: Short summary of the post.
---
```

## Tabele of Contents
목차(TOC)는 게시물의 오른쪽 패널에 표시된다. 전역적으로 끄려면 `_config.yml` 에서 `toc` 를 `false` 로 설정하면 된다.

## Comments
댓글을 활성화하기 위해서는 `_config.yml` 파일의 `comments` 를 설정하면 된다.

## Media

### URL Prefix
게시물에서 여러 리소스에 대해 중복된 URL 접두사를 정의해야 한다. 다음 두 매개변수를 설정하면 해결할 수 있다.
- CDN을 사용하여 미디어 파일을 호스팅하는 경우 `_config.yml` 에서 `cdn` 을 지정할 수 있다.
```yaml
cdn: https://cdn.com
```
 - 현재 게시물에 대한 리소스 경로 접두사를 지정하려면 게시물의 front matter에 `media_subpath` 를 설정하면 된다.
```yaml
---
media_subpath: /path/to/media/
---
```

### Images
#### Caption

이미지의 다음 줄에 이탤릭체를 추가하면 해당 줄이 캡션이 되어 이미지 맨 아래 나타난다.
```markdown
![img-description](/path/to/image)
_Image Caption_
```

#### Size
다음과 같이 이미지의 너비와 높이를 설정할 수 있다.
```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
```

#### Position
기본적으로 이미지는 중앙에 배치되지만 `normal`, `left`, `right` 를 사용하여 위치를 지정할 수 있다.
```markdown
![Desktop View](/assets/img/sample/mockup.png){: .normal }
```

#### Dark/Light mode
다음과 같이 다크 모드용, 라이트 모드용 이미지를 설정할 수 있다.
```markdown
![Light mode only](/path/to/light-mode.png){: .light }
![Dark mode only](/path/to/dark-mode.png){: .dark }
```

#### Shadow
다음과 같이 그림자 효과를 추가할 수 있다.
```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```

#### Preview Image
게시물 상단에 이미지를 추가하려면 1200 x 630 해상도 이미지를 추가해야 한다. 그렇지 않을 경우 이미지 크기가 조절되고 잘린다.
```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

### Video
다음 [링크](https://chirpy.cotes.page/posts/write-a-new-post/#video)를 참고하면 된다.

## Pinned Posts
게시물을 홈페이지 상단에 고정할 수 있으며, 고정된 게시물은 게시 날짜에 따라 역순으로 정렬된다.
```yaml
---
pin: true
---
```

## Prompts
프롬프트에는 `tip`, `info`, `warning`, `danger` 가 있다. 다음과 같이 블록 인용문에 `prompt-{type}` 을 추가하여 생성할 수 있다.

```Markdown
> Example line for prompt.
{: .prompt-info }
```

## Syntax
### Inline Code

```Markdown
`inline code`
```

### Filepath Highlight

```Markdown
`/path/to/a/file.extend`{: .filepath}
```

### Code Block

````markdown
```
Code Block
```
````

## Mathmatics
웹사이트 성능상의 이유로 수학 기능은 기본적으로 로드되지 않는다. 하지만 다음을 통해 활성화할 수 있다.
```yaml
---
math: true
---
```

다음 구문을 사용하여 수학 방정식을 추가할 수 있다.
- 블록으로 수학을 나타낼 때는 `$$ math $$` 로 추가하며 `$$` 앞뒤에 필수 빈 줄이 있어야 한다.
	- 방정식 번호 삽입은 `$$\begin{equation} math \end{equation}$$` 로 추가해야 한다.
	- 방정식 번호 참조는 방정식 블록에서 `\label{eq:label_name}` 을 사용하고 텍스트와 함께 인라인으로 `\eqref{eq:label_name}` 을 사용해야 한다.
- 인라인으로 수학을 나타낼 대는 `$$math$$` 로 추가하며 앞뒤에빈줄이 없어야 한다.

```markdown
<!-- Block math, keep all blank lines -->

$$
LaTeX_math_expression
$$

<!-- Equation numbering, keep all blank lines  -->

$$
\begin{equation}
  LaTeX_math_expression
  \label{eq:label_name}
\end{equation}
$$

Can be referenced as \eqref{eq:label_name}.

<!-- Inline math in lines, NO blank lines -->

"Lorem ipsum dolor sit amet, $$ LaTeX_math_expression $$ consectetur adipiscing elit."

<!-- Inline math in lists, escape the first `$` -->
1. \$$ LaTeX_math_expression $$
2. \$$ LaTeX_math_expression $$
3. \$$ LaTeX_math_expression $$
```

## Mermaid
다음과 같이 Mermaid를 활성화 할 수 있다.
```markdown
---
mermaid: true
---
```

## Reference
[https://chirpy.cotes.page/posts/write-a-new-post](https://chirpy.cotes.page/posts/write-a-new-post)
