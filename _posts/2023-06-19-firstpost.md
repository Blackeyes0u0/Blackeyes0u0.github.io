---
title : "Dimension Reduction"
excerpt : "md 파일에 마크다운 문법으로 작성하여 Github 원격 저장소에 업로드 해보자. 에디터는 Visual Studio code 사용! 로컬 서버에서 확인도 해보자. "

categories: 
  - dimension_reduction
  - image_embedding

tags:
  - [Linear_Algebra, Machine_Learning, Deep_Learning, Dimension_Reduction, Image_Embedding]
  
toc: true
toc_sticky: true

date: 2023-07-09
last_modified_at: 2023-07-09
---
## Dimension Reduction

papers : J.E.Hinton, R.R.Salakhutdinov, "Reducing the Dimensionality of Data with Neural Networks", Science, 2006

#### Abstract

- Dimensionality reduction is often used as a pre-processing step to improve the performance of machine learning algorithms.


we using the neural network to reduce the dimensionality of data.

\\(KL(P(x)||Q(x)) = \sum_{x} P(x) log \frac{P(x)}{Q(x)}\\)

\\(\vec{f_{ij,spring}} = -k(\vert\vec{x_i} - \vec{x_j}\vert - l_0){\vec{x_i} - \vec{x_j}\over{\vert\vec{x_i} - \vec{x_j}\vert}} \\)

\\(\begin{equation}
f_{i j, s p r i n g}=-k\left(\left|\vec{x}_i-\vec{x}_j\right|-l_0\right) \frac{\vec{x}_i-\vec{x}_j}{\left|\vec{x}_i-\vec{x}_j\right|}
\end{equation}\\)

\\(f_{i j, s p r i n g}=-k\left(\left|\vec{x}_i-\vec{x}_j\right|-l_0\right) \frac{\vec{x}_i-\vec{x}_j}{\left|\vec{x}_i-\vec{x}_j\right|}\\)

## PDF Document

<iframe src="../paper/dimension_reduction/sne.pdf" width="100%" height="800px">
  <p>Unable to display PDF. Click <a href="../paper/dimension_reduction/sne.pdf">here</a> to download it.</p>
</iframe>

byebye
---
# bye
<a href="../paper/dimension_reduction/sne.pdf" class="image fit"><img src="images/marr_pic.jpg" alt=""></a>
    