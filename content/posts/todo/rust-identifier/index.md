---
title: Create an identicon with Rust
subtitle:
date: 2026-05-25T23:45:38-06:00
slug: 7d94dc6
draft: true
description:
keywords:
comment: false
weight: 0
tags:
  - draft
  - rust
categories:
  - draft
  - programming
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
summary:
featuredImagePreview:
featuredImage:
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

<!--more-->

* **Identicon**

  : 1. An Identicon is a visual representation of a hash value, usually of an IP address, that serves to identify a user of a computer system as an effectively random form of avatar while protecting the user's privacy. The original Identicon was a 9-block graphic, and the representation has been extended to other graphic forms by third parties.

## Description

Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum. Lorem Ipsum.

![identicon example](images/identicon-wako.png "Picture 1.1: 'wako' represented as an identicon.")

Hash: Lorem upsum..

If you are interested in check your github's identicon, you can use this site:

[https://github.com/identicons/\<username\>.png](https://github.com/identicons/4k1k0.png)

```shell
18556113149732110175237116169101016523939
```

| decimal | hex |
|:-:|:-:|
| 185 | b9 |
| 56 | 38 |
| 113 | 71 |
| 149 | 95 |
| 73 | 49 |
| 21 | 15 |
| 10 | 0a |
| 175 | af |
| 237 | ed |
| 116 | 74 |
| 169 | a9 |
| 10 | 0a |
| 10 | 0a |
| 165 | a5 |
| 239 | ef |
| 39 | 27 |

Take the first 3 numbers to get the color:

| base | r | g | b |
|:-:|:-:|:-:|:-:|
| decimal | 185 | 56 | 113 |
| hex | b9 | 38 | 71 |

[![#b93871](images/base-color.png "Picture 1.2 Color #b93871")](https://www.colorhexa.com/b93871)

## More info

```shell
[185, 56, 113, 149, 73, 21, 10, 175, 237, 116, 169, 10, 10, 165, 239, 39]
[
  (185, 56, 113),
  (149, 73, 21),
  (10, 175, 237),
  (116, 169, 10),
  (10, 165, 239),
]

[
  (185, 56, 113, 56, 185),
  (149, 73, 21, 73, 149),
  (10, 175, 237, 175, 10),
  (116, 169, 10, 169, 116),
  (10, 165, 239, 165, 10),
]

[185, 56, 113, 56, 185, 149, 73, 21, 73, 149, 10, 175, 237, 175, 10, 116, 169, 10, 169, 116, 10, 165, 239, 165, 10]


pub type Point (u8, usize);

[
(185, 0),
(56, 1),
(113, 2),
(56, 3),
(185, 4),
(149, 5),
(73, 6),
(21, 7),
(73, 8),
(149, 9),
(10, 10),
(175, 11),
(237, 12),
(175, 13),
(10, 14),
(116, 15),
(169, 16),
(10, 17),
(169, 18),
(116, 19),
(10, 20),
(165, 21),
(239, 22),
(165, 23),
(10, 24)
]


[
(56, 1),
(56, 3),
(10, 10),
(10, 14),
(116, 15),
(10, 17),
(116, 19),
(10, 20),
(10, 24)
]

Gen pixel map:


pixel_map:
[
  ((50, 0), (100, 50)),
  ((150, 0), (200, 50)),
  ((0, 100), (50, 150)),
  ((200, 100), (250, 150)),
  ((0, 150), (50, 200)),
  ((100, 150), (150, 200)),
  ((200, 150), (250, 200)),
  ((0, 200), (50, 250)),
  ((200, 200),(250, 250))
]

```

## Info

```shell
[185, 56, 113, 149, 73, 21, 10, 175, 237, 116, 169, 10, 10, 165, 239, 39]
----------------------------------------
Color { r: 185, g: 56, b: 113 }
----------------------------------------
"wako"
----------------------------------------
"images/18556113149732110175237116169101016523939.png"
----------------------------------------
grid: [(56, 1), (56, 3), (10, 10), (10, 14), (116, 15), (10, 17), (116, 19), (10, 20), (10, 24)]
----------------------------------------
pixel_map: [((50, 0), (100, 50)), ((150, 0), (200, 50)), ((0, 100), (50, 150)), ((200, 100), (250, 150)), ((0, 150), (50, 200)), ((100, 150), (150, 200)), ((200, 150), (250, 200)), ((0, 200), (50, 250)), ((200, 200), (250, 250))]
----------------------------------------
R:185 | G:56 | B:113
```

## References

- [Stack Overflow](https://stackoverflow.com/questions/65122991/official-github-identicon-algorithm)
- [a](https://google.com)
- [a](https://google.com)
