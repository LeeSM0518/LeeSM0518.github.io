---
title: Text and Typography
date: 2024-11-07 19:00:00 +0900
categories: blog
tags:
  - blog
description: Examples of text, typography, math equations, diagrams, flowcharts, pictures, videos, and more.
math: "true"
mermaid: "true"
---
## Headings

```markdown
# H1 - heading
## H2 - heading
### H3 - heading
#### H4 - heading
```

# H1 - heading
## H2 - heading
### H3 - heading
#### H4 - heading

---

## Paragraph

Quisque egestas convallis ipsum, ut sollicitudin risus tincidunt a. Maecenas interdum malesuada egestas. Duis consectetur porta risus, sit amet vulputate urna facilisis ac. Phasellus semper dui non purus ultrices sodales. Aliquam ante lorem, ornare a feugiat ac, finibus nec mauris. Vivamus ut tristique nisi. Sed vel leo vulputate, efficitur risus non, posuere mi. Nullam tincidunt bibendum rutrum. Proin commodo ornare sapien. Vivamus interdum diam sed sapien blandit, sit amet aliquam risus mattis. Nullam arcu turpis, mollis quis laoreet at, placerat id nibh. Suspendisse venenatis eros eros.

---

## Lists

### Ordered list

```markdown
1. Firstly
2. Secondly
3. Thirdly
```

1. Firstly
2. Secondly
3. Thirdly


### Unorderd list

```markdown
- Chapter
  - Section
    - Paragraph
```

- Chapter
	- Section
		- Paragraph


### ToDo List

```markdown
- [ ] Job
  - [x] Step 1
  - [x] Step 2
  - [ ] Step 3
```

- [ ] Job
	- [x] Step 1
	- [x] Step 2
	- [ ] Step 3


### Description list

```markdown
Sun
: the star around which the earth orbits

Moon
: the natural satellite of the earth, visible by reflected light from the sun
```

Sun
: the star around which the earth orbits

Moon
: the natural satellite of the earth, visible by reflected light from the sun

---

## Block Quote

```markdown
> This line shows the _block quote_.
```

> This line shows the _block quote_.

---

## Prompts

```markdown
> An example showing the `tip` type prompt.
{: .prompt-tip }

> An example showing the `info` type prompt.
{: .prompt-info }

> An example showing the `warning` type prompt.
{: .prompt-warning }

> An example showing the `danger` type prompt.
{: .prompt-danger }
```

> An example showing the `tip` type prompt.
{: .prompt-tip }

> An example showing the `info` type prompt.
{: .prompt-info }

> An example showing the `warning` type prompt.
{: .prompt-warning }

> An example showing the `danger` type prompt.
{: .prompt-danger }

---

## Tables

```markdown
| Company                      | Contact          | Country |
| :--------------------------- | :--------------- | ------: |
| Alfreds Futterkiste          | Maria Anders     | Germany |
| Island Trading               | Helen Bennett    |      UK |
| Magazzini Alimentari Riuniti | Giovanni Rovelli |   Italy |
```

| Company                      | Contact          | Country |
| :--------------------------- | :--------------- | ------: |
| Alfreds Futterkiste          | Maria Anders     | Germany |
| Island Trading               | Helen Bennett    |      UK |
| Magazzini Alimentari Riuniti | Giovanni Rovelli |   Italy |

---

## Links

```markdown
<http://127.0.0.1:4000>
```

<http://127.0.0.1:4000>

---

## Footnote

```markdown
Click the hook will locate the footnote[^footnote], and here is another footnote[^fn-nth-2].
```

Click the hook will locate the footnote[^footnote], and here is another footnote[^fn-nth-2].

---

## Inline code

```markdown
This is an example of `Inline Code`.
```

This is an example of `Inline Code`.

---

## Filepath

```markdown
Here is the `/path/to/the/file.extend`{: .filepath}.
```

Here is the `/path/to/the/file.extend`{: .filepath}.

---

## Code blocks

### Common

````markdown
```text
This is a common code snippet, without syntax highlight and line number.
```
````

```text
This is a common code snippet, without syntax highlight and line number.
```

### Specific Language

````bash
```bash
if [ $? -ne 0 ]; then
  echo "The command was not successful.";
  #do the needful / exit
fi;
```
````

```bash
if [ $? -ne 0 ]; then
  echo "The command was not successful.";
  #do the needful / exit
fi;
```

### Specific filename

````markdown
```sass
@import
  "colors/light-typography",
  "colors/dark-typography";
```
{: file='_sass/jekyll-theme-chirpy.scss'}
````

```sass
@import
  "colors/light-typography",
  "colors/dark-typography";
```
{: file='_sass/jekyll-theme-chirpy.scss'}

---

## Mathmatics

```markdown
$$
\begin{equation}
  \sum_{n=1}^\infty 1/n^2 = \frac{\pi^2}{6}
  \label{eq:series}
\end{equation}
$$

We can reference the equation as \eqref{eq:series}.

When $a \ne 0$, there are two solutions to $ax^2 + bx + c = 0$ and they are

$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $$
```

$$
\begin{equation}
  \sum_{n=1}^\infty 1/n^2 = \frac{\pi^2}{6}
  \label{eq:series}
\end{equation}
$$

We can reference the equation as \eqref{eq:series}.

When $a \ne 0$, there are two solutions to $ax^2 + bx + c = 0$ and they are

$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $$

---

## Mermaid SVG

````markdown
```mermaid
 gantt
  title  Adding GANTT diagram functionality to mermaid
  apple :a, 2017-07-20, 1w
  banana :crit, b, 2017-07-23, 1d
  cherry :active, c, after b a, 1d
```
````

```mermaid
 gantt
  title  Adding GANTT diagram functionality to mermaid
  apple :a, 2017-07-20, 1w
  banana :crit, b, 2017-07-23, 1d
  cherry :active, c, after b a, 1d
```

---

## Images

### Default (with caption)

```markdown
![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" }
_Full screen width and center alignment_
```

![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" }
_Full screen width and center alignment_

### Left aligned

```markdown
![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-75 .normal}
```

![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-75 .normal}

### Float to left

```markdown
![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-50 .left}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.
```

![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-50 .left}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.

### Float to right

```markdown
![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-50 .right}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.
```

![Desktop View](/assets/img/20241107-full-screen.png){: width="972" height="589" .w-50 .right}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.

### Dark/Light mode & Shadow

```markdown
![light mode only](/assets/img/20241107-white-mode.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark mode only](/assets/img/20241107-dark-mode.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }
```

![light mode only](/assets/img/20241107-white-mode.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark mode only](/assets/img/20241107-dark-mode.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }

---

## Video

```markdown
{% include embed/youtube.html id='Balreaj8Yqs' %}
```

{% include embed/youtube.html id='Balreaj8Yqs' %}

---

## Reverse Footnode

```markdown
[^footnote]: The footnote source
[^fn-nth-2]: The 2nd footnote source
```

[^footnote]: The footnote source
[^fn-nth-2]: The 2nd footnote source
