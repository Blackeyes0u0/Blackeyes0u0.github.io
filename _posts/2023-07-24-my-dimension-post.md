---
title : "Dimension Reduction"
excerpt : "차원 축소에 대한 내용을 담고 있고, linear dimension reduction은 PCA, non-linear dimension reduction는 VQA를 통해 설명 하였다.. "

categories: 
  - dimension_reduction
  - image_embedding

tags:
  - [Linear_Algebra, Machine_Learning, Deep_Learning, Dimension_Reduction, Image_Embedding]
  
toc: true
toc_sticky: true

date: 2023-07-24
last_modified_at: 2023-07-24
---

papers : J.E.Hinton, R.R.Salakhutdinov, "Reducing the Dimensionality of Data with Neural Networks", Science, 2006

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0001.jpg?raw=true)

#### Abstract

어떤 task를 함에 있어서, 제공하는 data는 우리가 활용하기 적합한 차원과 분포를 띄고 있는가? 라는 물음을 시작으로 차원 축소에 대해서 소개해 보았습니다.
보시다가 궁금증이나 틀린부분은 댓글로 달아주시면 감사하겠습니다.


#### Intro

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0002.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0003.jpg?raw=true)

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0004.jpg?raw=true)


#### About Data

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0005.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0006.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0007.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0008.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0009.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0010.jpg?raw=true)

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0011.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0012.jpg?raw=true)

#### PCA(Principal component analysis)

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0013.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0014.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0015.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0016.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0017.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0018.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0019.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0020.jpg?raw=true)

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0021.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0022.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0023.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0024.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0025.jpg?raw=true)

#### Autoencoder

![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0026.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0027.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0028.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0029.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0030.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0031.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0032.jpg?raw=true)
![image](https://github.com/Blackeyes0u0/Dimension_Reduction_and_Reconstruction/blob/main/images/TechTalkCFP_dimension_reduction_page-0033.jpg?raw=true)

#### Reference

[colah Visualizing MNIST BLOG](https://colah.github.io/posts/2014-10-Visualizing-MNIST/​)

[Auto-Encoding Variational Bayes](https://arxiv.org/abs/1312.6114)​​
: Diederik P Kingma, Max Welling, arxiv

[A Tutorial on Principal Component Analysis](​​https://arxiv.org/abs/1404.1100​​)
: Jonathon Shlens, arxiv

[Hands On ML books github](https://github.com/ExcelsiorCJH/Hands-On-ML/​​)

[이활석님의 Autoencoder](https://www.slideshare.net/NaverEngineering/ss-96581209​​)

[scikit-learn manifold](https://scikit-learn.org/stable/modules/manifold.html#manifold​​)

[AAILab Kaist youtube ](https://www.youtube.com/@aailabkaist6236/playlists​​)

[naver d2](https://www.youtube.com/@naverd2848/playlists​​)

[SNE](​https://proceedings.neurips.cc/paper/2002/hash/6150ccc6069bea6b5716254057a194ef- Abstract.html​)
: Geoffrey E. Hinton, Sam Roweis

[t-SNE](https://www.jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf?fbcl)
: Geoffrey E. Hinton


​
## PDF Document

[PDF URL](https://github.com/Blackeyes0u0/Blackeyes0u0.github.io/blob/main/paper/Dimension/TechTalkCFP_dimension_reduction.pdf)

<!-- 
<iframe src="../paper/Dimension/TechTalkCFP_dimension_reduction.pdf" width="100%" height="800px" type="application/pdf"></iframe> -->

thanks to visit