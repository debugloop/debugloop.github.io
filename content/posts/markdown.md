---
title: All the Markdown Features I'm ever likely to use
date: 2020-03-24T15:31:13+01:00
tags:
  - markdown
  - latex
---

{{< lead >}}
Some general _bla bla_ about the article.
{{< /lead >}}
This is part of the summary presented in the post list. Inline Code `here`.
There should be quite a bit of content so that the summary in the list view of
posts does not include stranger things from down below. To his end, here some
Lorem Ipsum: Excepteur sint occaecat cupidatat non proident, sunt in culpa qui
officia deserunt mollit anim id est laborum. Links work too:
[Wikipedia](https://wikipedia.com).

Some math for testing:
{{< katex >}}

$$
\frac{n!}{k!(n-k)!} = \binom{n}{k}
$$

## A paragraph (with a footnote)

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
incididunt ut labore et dolore __magna aliqua__. Ut enim ~~ad minim~~ veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum. That's
"some" text with a footnote.[^0]

{{< mermaid >}}
graph LR;
A[Lemons]-->B[Lemonade];
B-->C[Profit]
{{< /mermaid >}}


[^0]: "Some", because I wanted to test the quotation marks.

Quotes are nice:
> Everything we hear is an opinion, not a fact.
> Everything we see is a perspective, not the truth.
>
> &emsp;--- not Marcus Aurelius, actually

Some code with highlighted return lines:

```python {hl_lines=[3,7]}
def __getattribute__(self, name):
    if name == 'insert' and self.__root:
        return object.__getattribute__(self, name)
    elif name == 'insert' and not self.__root:
        print("This is not a root node.")
        raise AttributeError
    else:
        return object.__getattribute__(self, name)
```

{{< button href="https://go.dev/play" target="_blank" >}}
run on playground
{{< /button >}}

## Second level header, the other is reserved for the top spot
A list, but first some general bla bla to get some kind of paragraph going in
this place. If it looks to empty, the following list will just hang around in
place, looking weird:
* bla
* bla
* blah

{{< alert "twitter" >}}
Don't forget to [follow me](https://twitter.com/debugloop) on Twitter.
{{< /alert >}}

### Third level header (I'll never use fourth...)
Lorem ipsum dolor sit amet consetetur sadipscing elitr, sed diam nonumy eirmod
tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At
vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd
gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum
dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor
invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero
eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no
sea takimata sanctus est Lorem ipsum dolor sit amet:

1. bla
1. bla
2. blah

{{< chart >}}
type: 'bar',
data: {
  labels: ['bla', 'blargh', 'blah', 'blabla', 'bubba'],
  datasets: [{
    label: '# of votes',
    data: [12, 19, 3, 5, 2, 3],
  }]
}
{{< /chart >}}





And finally, a todo list:
* [x] check all features
* [x] make mathjax and fonts load locally
* [ ] profit??

## Tables, whoo hoo

 Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|Species
------------:|-----------:|------------:|-----------:|:-------
          5.1|         3.5|          1.4|         0.2|setosa
          4.9|         3.0|          1.4|         0.2|setosa
          4.7|         3.2|          1.3|         0.2|setosa
          4.6|         3.1|          1.5|         0.2|setosa
          5.0|         3.6|          1.4|         0.2|setosa
          5.4|         3.9|          1.7|         0.4|setosa

## Thats it, looking good?
![thumbs up](https://i.imgur.com/4PcPCIJ.gif)
