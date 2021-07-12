---
title: Teste
author: Santhiago Cristiano
date: '2021-07-12'
slug: []
categories: []
tags: ["R"]
type: ''
subtitle: ''
image: ''
---




Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla sit amet ex eu lectus sodales tincidunt non eu magna. Proin dapibus libero sit amet erat semper, id consequat metus condimentum. Fusce cursus nec neque eu vulputate. Curabitur tempor sit amet nunc eget malesuada. Integer in luctus sapien, eget blandit sem. Aliquam blandit, tortor vel rhoncus molestie, nisi ipsum interdum orci, quis pharetra tellus libero sed arcu. Sed dapibus massa at lacus vulputate placerat.


```r
ggplot2::ggplot(mtcars, ggplot2::aes(factor(cyl), mpg)) +
  ggplot2::geom_point()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Sed sit amet velit et leo interdum malesuada. Donec cursus iaculis arcu, nec pulvinar velit posuere quis. Sed a risus leo. Donec accumsan erat vitae congue faucibus. Morbi in dui maximus, porttitor sem vel, rutrum enim. Fusce blandit dictum ante ut ultrices. Sed tristique facilisis arcu, a cursus orci consectetur nec. Nam scelerisque pellentesque metus, et egestas lectus malesuada nec. Phasellus sit amet metus in odio mattis lobortis. Fusce sed tortor non magna dapibus elementum nec vitae nisi.

Nulla sollicitudin purus ex, at pharetra odio tristique fringilla. Sed at lacus sit amet magna cursus venenatis sed eget ligula. Aliquam commodo ac lorem rutrum iaculis. Nulla facilisi. Integer quis congue dui. Vestibulum ut magna sem. Vivamus luctus rutrum ante, quis rhoncus ipsum bibendum a. Ut nec massa in diam blandit tristique. Vivamus in enim iaculis, fermentum mauris sed, luctus lectus. Sed et mauris lobortis, feugiat neque id, finibus nibh. Vivamus sed nisl rhoncus, ultrices turpis ac, lobortis massa. Donec fermentum sem eleifend vehicula ornare. Proin convallis orci maximus interdum semper. Curabitur lacinia diam a dolor sagittis consectetur.


```r
oplot <- ggplot2::ggplot(
  Orange,
  ggplot2::aes(x = age, y = circumference, colour = Tree)
) +
  ggplot2::geom_point() +
  ggplot2::geom_line() +
  ggplot2::guides(colour = "none") +
  ggplot2::theme_bw()
oplot
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Morbi ipsum tortor, ultricies a erat sed, varius placerat lectus. Integer porttitor diam a posuere bibendum. Nam venenatis orci sed cursus hendrerit. Maecenas rhoncus urna non sem porttitor sodales. Sed nec erat sed magna tempor condimentum. In a porta quam, nec elementum magna. Sed molestie volutpat felis, sit amet vulputate ipsum cursus ac. Vestibulum bibendum nisl in nisl porttitor eleifend. Nulla suscipit, dolor quis consequat porta, mauris justo sagittis ligula, eget congue mauris risus sed turpis. Vestibulum tristique, libero vitae porta feugiat, eros nisi bibendum ex, id volutpat erat enim ut nulla. Duis quis elementum mi, ut mattis quam. Suspendisse suscipit consequat euismod. Aliquam in condimentum nibh. Phasellus eu ante quis dui rutrum lobortis.


```r
iris_2 <- tidyr::pivot_longer(iris, -Species, names_to = "type", values_to = "value")
iris_2
```

```
## # A tibble: 600 x 3
##    Species type         value
##    <fct>   <chr>        <dbl>
##  1 setosa  Sepal.Length   5.1
##  2 setosa  Sepal.Width    3.5
##  3 setosa  Petal.Length   1.4
##  4 setosa  Petal.Width    0.2
##  5 setosa  Sepal.Length   4.9
##  6 setosa  Sepal.Width    3  
##  7 setosa  Petal.Length   1.4
##  8 setosa  Petal.Width    0.2
##  9 setosa  Sepal.Length   4.7
## 10 setosa  Sepal.Width    3.2
## # ... with 590 more rows
```


Vivamus lobortis, urna dapibus mollis vestibulum, neque mauris condimentum est, sit amet tempus turpis arcu nec turpis. Nulla posuere eros quis velit ultrices, vitae laoreet ligula molestie. Pellentesque leo tellus, eleifend eget orci et, vestibulum blandit diam. Vivamus consectetur, nunc sollicitudin posuere eleifend, ex nisl sodales risus, vel feugiat dolor libero mattis massa. Nulla in malesuada magna. Curabitur pretium egestas lorem, et facilisis mi mattis a. Duis tincidunt neque nec est sollicitudin efficitur. Sed purus mi, pellentesque ac eleifend quis, pulvinar non dui.


```r
iris_2 %>% 
  ggplot2::ggplot() + 
  ggplot2::aes(Species, value, color = type) +
  ggplot2::geom_point(position = ggplot2::position_jitter()) +
  ggplot2::theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

