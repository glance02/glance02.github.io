# Optimization over Disentangled Encoding: Unsupervised Cross-Domain Point Cloud Completion via Occlusion Factor Manipulation

Jingyu Gong<sup>1</sup>, Fengqi Liu<sup>1</sup>, Jiachen Xu<sup>1</sup>, Min Wang<sup>2</sup>, Xin Tan<sup>3</sup>, Zhizhong Zhang<sup>3</sup>, Ran Yi<sup>1</sup>, Haichuan Song<sup>3</sup>, Yuan Xie<sup>3(B)</sup>, and Lizhuang Ma<sup>1,3,4(B)</sup> 

<sup>1</sup> Shanghai Jiao Tong University, Shanghai, China gongjingyu,liufengqi,xujiachen,ranyi @sjtu.edu.cn 2 SenseTime Research, Shanghai, China wangmin@sensetime.com 

3 East China Normal University, Shanghai, China tanxin2017@sjtu.edu.cn, zzzhang,hcsong,yxie @cs.ecnu.edu.cn 4 Qing Yuan Research Institute, SJTU, Shanghai, China ma-lz@cs.sjtu.edu.cn 

Abstract. Recently, studies considering domain gaps in shape completion attracted more attention, due to the undesirable performance of supervised methods on real scans. They only noticed the gap in input scans, but ignored the gap in output prediction, which is specific for completion. In this paper, we disentangle partial scans into three (domain, shape, and occlusion) factors to handle the output gap in cross-domain completion. For factor learning, we design view-point prediction and domain classification tasks in a self-supervised manner and bring a factor permutation consistency regularization to ensure factor independence. Thus, scans can be completed by simply manipulating occlusion factors while preserving domain and shape information. To further adapt to instances in the target domain, we introduce an optimization stage to maximize the consistency between completed shapes and input scans. Extensive experiments on real scans and synthetic datasets show that ours outperforms previous methods by a large margin and is encouraging for the following works. Code is available at https://github.com/ azuki-miho/OptDE. 

Keywords: Point cloud completion Cross-domain Disentanglement 

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/a20cf7d751d6a855812289e7a9d9723ca686af343d2be7e06b955b81aca522de.jpg)



Fig. 1. (a) presents the domain gap between objects from the same category but different datasets where topology and geometry patterns vary a lot, as well as the feature distribution. (b) illustrates the output domain gap which is specific for completion task in contrast to cross-domain classification. In (c), we disentangle any partial scan into three independent factors, and completion can be simply implemented by setting occlusion factors to zero vector (red arrow) while well preserve shape and domain features. (Color figure online)


## 1 Introduction

Shape completion in which we infer complete shapes given partial ones, attracted lots of attention recently, for its wide application in robotics, path planning and VR/AR [6,14,42]. However, real scan completion is quite challenging due to the irregularity of point cloud and absence of complete real shapes for training. 

Previous methods [15,30,35] had widely exploited completion on virtual 3D models like ShapeNet [5] and achieved desirable performance. That is attributed to the availability of complete shapes, and corresponding partial point clouds can be obtained through virtual scanning [41]. However, it is hard to use supervised methods for real point cloud completion, because complete shapes of real objects are usually unavailable for supervision. Meanwhile, completion networks trained on virtual 3D model are commonly not able to generalize well to infer complete real objects, especially when there are large domain gaps between the synthesized shapes and real objects [6]. So, the key for recovering real scans is to handle the cross-domain completion, where geometry and topology of objects from the same category are diferent between various datasets as shown in Fig. 1(a). 

To alleviating the influence of domain gaps, pcl2pcl [6] trained two autoencoders and an adaptation network to transform the latent code of a real scan to that of complete virtual one for each category. Then, the decoder for complete virtual shapes can map the transformed codes into complete shapes. However, they ignored a serious problem which is specific for completion task. In contrast to cross-domain shape classification whose output space (i.e., categories) is invariant to diferent input domains, the predicted complete point cloud should correspond to the domain of input partial shapes (see Fig. 1(b)). That means an output domain gap in completion task, as illustrated by the domain factor distribution ofcomplete shapes from CRN [32] and ModelNet [38] after t-SNE in Fig. 1 (a). Therefore, decoder trained to infer complete virtual shape cannot complete real scans. Recently, based on TreeGAN [27] and GAN Inversion [22], ShapeInversion [42] fine-tuned the decoder trained on virtual shapes during optimization stage for better performance. The underlying principle is that, fine-tuning according to real scans could adapt the decoder to real instances, alleviating output domain gap to some extent. However, adaptation in optimization alone is not enough, especially when the output domain gaps are quite large. 

To handle the output domain gap, we assume the category of input partial shapes is known in advance as previous works [6,33,42], and introduce an intensively disentangled representation for partial shapes of each category via comprehensive consideration of both shape completion and output domain gap. Specifically, in our assumption, there are three generative factors, i.e., occlusion, domain and domain-invariant shape factor, underlying a given partial cloud as shown in Fig. 1(c). For complete point clouds from the same category but different domains, they usually share the same semantic parts but quite diferent topology and geometry patterns. Thus, we attempt to disentangle any complete shape into a domain factor and a domain-invariant shape factor. While for a given complete shape, the partial point clouds generated from scanning vary a lot due to the occlusion caused by diferent scanning view-point [41]. So, we also introduce an occlusion factor which indicates the view-point for partial scans. 

Based on this assumption, any partial scan can be disentangled into these factors no matter it is virtual or real, and we also assume the occlusion factor will be all-zero if the input shape is complete. Thus, point cloud completion can be simply implemented by manipulating the occlusion factor (setting to zero vector, see the occlusion factor manipulation in Fig. 1(c)), while shape and domain information can be well-preserved in the output prediction. 

To thoroughly explore these three factors, we design several components to facilitate the disentanglement. (a) It is noteworthy that occlusion in partial shapes is usually caused by scanning at fixed view-point. Therefore, we introduce a self-supervised view-point prediction task for better occlusion factor/feature learning. (b) We take a domain discriminator judging whether a shape is virtual or real to extract domain factor, and employ another domain discriminator to decouple the domain information from the shape feature in an adversarial way. Additionally, we utilize the completion task to ensure the domain and domain-invariant shape factors are enough to infer complete shapes. (c) Inspired by PMP [10], for more intensive disentanglement, we derive a new factor permutation consistency regularization by randomly swapping the factors between samples, and introduce an inverse structure of auto-encoder, decoderencoder for swapped factor reconstruction to ensure these factors independent to each other. 

To further adapt our prediction to each partial instance in the target domain, we embrace a collaboration of regression and optimization. We use the trained encoder to obtain the disentangled factors of input cloud and set the occlusion factor to zero. Then, the combined factors can give a good initial prediction given the decoder. Later, Chamfer Distance between partial input and masked prediction [42] is used to fine-tune these factors and decoder within several iterations. Thanks to the collaboration, our method can give prediction 100 faster than pure optimization method like [42] and achieve much better performance. 

To evaluate the performance on cross-domain completion, we test on real datasets ScanNet [8], MatterPort3D [4] and KITTI [12] like previous works [6,42]. We also utilize two additional point cloud completion datasets 3D-FUTURE [9] and ModelNet [38] with complete shapes for more comprehensive evaluation. The experimental results demonstrate our method can well cover the gap in output prediction and significantly improve cross-domain completion performance. 

## 2 Related Works

Point Cloud Completion. Inspired by PointNet [25], PCN [41] designed an auto-encoder with folding operation [40] for shape completion. Later, a great performance boost has been brought in virtual shape completion, where paired shapes are available for training [20,23,30,34,35,39,43]. To generalize to real scans, pcl2pcl [6] trained two auto-encoders for virtual complete shapes and real partial scans, and an adaptation network to map the latent codes of real scans to that of virtual complete shapes. Cycle4Completion [33] added a reverse mapping function to maintain shape consistency. Meanwhile, ShapeInversion [42] searched for the latent code that best reconstruct the shapes during the optimization stage, given a generator for complete virtual shapes. Even though they finetuned the generator for better adaptation of partial scans, it is far from enough when the domain gaps are large. 

Compared with these methods, we attempt to handle the domain gaps in the output prediction, which is specific for generation or completion task. In our method, we disentangle occlusion factor and domain factor for better completion of real scans while preserving domain-specific patterns. 

Disentangled Representation Learning. Disentangled factor learning was explored under the concept of “discovering factorial codes” [2] by minimizing the predictability of one factor given remaining units [26]. Based on VAE [17], FactorVAE proposed to disentangle the latent code to be factorial and independent [16]. Factors can be more easily used for image manipulation and translation after disentanglement [19,21]. To reduce the domain discrepancy, domain-specific and domain-invariant features were disentangled for better recognition performance [24,37]. To ensure the disentanglement, additional constraints on factor through recombination were used in motion and 3D shape modeling [1,7]. 

Inspired by their works, we design a disentangled space consisting of occlusion, domain and domain-invariant shape factors for cross-domain shape completion where complete shapes in target domain are unavailable for training. 

Collaboration of Regression and Optimization. Methods based on optimization and regression correspond to generative and discriminative models respectively, and recently their cooperation became prevailing for its speed and performance. In pose estimation, optimization was used on a good initial pose estimated by a regression approach [28]. Later, results obtained by optimization further supervised the regression network [18]. To speed up the process of finding latent code z of the generator that best reconstruct an image in image manipulation, an encoder is used for good initialization before further optimization [3]. 

Inspired by these methods, we further optimize the prediction given by disentangled encoding within several iterations to adapt to each input partial instance in the target domain, making it 100 faster than optimization-based completion method and achieve much better performance. 

## 3 Method

Overview. The framework of our two-stage method is shown in Fig. 2. In the first stage, we attempt to disentangle the input partial point cloud (unpaired source domain partial shapes generated from complete ones through real-time rendering and target domain partial scans) into three factors, naming view-based occlusion factor, domain factor, and domain-invariant shape factor as shown in Fig. 2(a). Here, we design a view-point prediction task in a self-supervised manner to disentangle the occlusion information caused by scanning. Concurrently, domain discriminators are taken to disentangle the domain-specific information from domain-invariant shape features. Later, three factors will be combined to output reconstructed partial point clouds. Meanwhile, we can simply predict the complete shapes by setting the occlusion factor to zero vector. Additionally, the independence of these three factors are ensured by randomly permuting the factors within a batch and keeping combined factors consistent after a decoderencoder structure as presented by Fig. 2(b). In the second stage, for completion of specific partial point clouds, we optimize the disentangled factors and decoder obtained from stage 1 within several iterations to better adapt to each input partial point cloud instance (Fig. 2(c)). 

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/6d0dea8aed7be32abf5f0695e3e1003bf24fee54c9d16ff6a3a3102d9b8c8ca2.jpg)



Fig. 2. Overall framework of Optimization over Disentangled Encoding. (a) shows the supervision given by view-point prediction, domain discrimination, reconstruction and completion. (b) shows the procedure of factor permutation consistency where factors are more intensively disentangled. Here, the Encoder, Decoder and Disentanglers are shared with (a). (c) shows the optimization procedure over disentangled factors of completed partial shapes and the Decoder is initialized by pre-trained model from (a).


## 3.1 Disentangled Representation for Completion

According to the disentanglement assumption [10,13], there are intrinsic factors $\{ f _ { i } \} _ { i = 1 } ^ { l }$ in much lower dimension that generate the observed samples in high dimension point cloud space $\mathcal { P } ( f _ { 1 } , \cdots , f _ { l } )$ ). In our method, we attempt to disentangle the common partial point cloud into three independent factors, including the occlusion factor $f _ { o } ,$ , domain factor $f _ { d }$ , and domain-invariant shape factor $f _ { s }$ 

View-Based Occlusion Factor. Partial shapes are mainly caused by occlusion, since a complete shape will generate various partial clouds when scanned from diferent view-points. So, scanning view-point plays a key role in point cloud completion and we aim to disentangle the view-based occlusion factor specially [31]. 

1) View-point Prediction: To disentangle the view-based occlusion factor, we design a view-point prediction task in a self-supervised manner. Here, we assume the view-point is located in a unit sphere. As shown in Fig. 2(a), we first randomly generate azimuth and elevation angles $( \rho , \theta )$ as the view-point direction, and rotate the complete point cloud accordingly. Then, based on zbufer [29], we design a real-time implementation to render the complete shapes in a fast non-diferentiable way and obtain the partial clouds in source domain. 

Then, the generated partial shapes will be fed into the shared encoder to extract common features, and a specific disentangler is utilized to obtain occlusion factor $f _ { o }$ . For better factor learning, we introduce a view-point predictor module VP, consisting of several MLPs, to predict the view-point of point cloud $( \hat { \rho } , \hat { \theta } ) = V P ( f _ { o } )$ . The loss for view-point prediction can be formulated as follows: 

$$
\mathcal {L} _ {v p} = (\rho - \hat {\rho}) ^ {2} + (\theta - \hat {\theta}) ^ {2}.\tag{1}
$$

Here, we choose to predict the azimuth and elevation angle directly rather than the rotation matrix due to their independence and simplicity. 

2) Occlusion Factor Manipulation: Additionally, we assume it is the occlusion factor that makes the point cloud incomplete, and the decoder will predict the complete shape when the occlusion factor is zeroed out. So, for the same input partial point cloud, it can generate reconstructed partial shapes and complete objects by simply manipulating the occlusion factor. The corresponding latent factors for reconstructed partial shapes $\left( z _ { p } \right)$ and complete ones $\left( z _ { c } \right)$ are: 

$$
z _ {p} = f _ {o} \otimes f _ {d} \otimes f _ {s}, z _ {c} = \mathbf {0} \otimes f _ {d} \otimes f _ {s},\tag{2}
$$

where $\otimes$ indicates vector concatenation. Therefore, the reconstructed partia shape and completed shape are $\hat { \mathcal { P } } _ { p } = D e c ( z _ { p } )$ and $\hat { \mathcal { P } } _ { c } = D e c ( z _ { c } )$ , respectively, where $D e c ( \cdot )$ is the decoder. Chamfer Distance (CD) or Unidirectional Chamfer Distance (UCD) between the output prediction and target shapes are used to supervise the whole network. We take the form of CD as previous works [23,42]: 

$$
\mathcal {C D} (\mathcal {P} _ {1}, \mathcal {P} _ {2}) = \frac {1}{| \mathcal {P} _ {1} |} \sum_ {p _ {1} \in \mathcal {P} _ {1}} \min _ {p _ {2} \in \mathcal {P} _ {2}} \| p _ {1} - p _ {2} \| _ {2} ^ {2} + \frac {1}{| \mathcal {P} _ {2} |} \sum_ {p _ {2} \in \mathcal {P} _ {2}} \min _ {p _ {1} \in \mathcal {P} _ {1}} \| p _ {1} - p _ {2} \| _ {2} ^ {2},\tag{3}
$$

and UCD is formulated as follows: 

$$
\mathcal {U C D} (\mathcal {P} _ {1}, \mathcal {P} _ {2}) = \frac {1}{| \mathcal {P} _ {1} |} \sum_ {p _ {1} \in \mathcal {P} _ {1}} \min _ {p _ {2} \in \mathcal {P} _ {2}} \| p _ {1} - p _ {2} \| _ {2} ^ {2}.\tag{4}
$$

Here, we use $\mathcal { C D } ( \cdot )$ to supervise the reconstructed partial shape from source $\hat { \mathcal { P } } _ { p } ^ { s }$ and target domains $\hat { \mathcal { P } } _ { p } ^ { t } ,$ , and predicted complete shape from source domains $\mathcal { \hat { P } } _ { c } ^ { s }$ For inferred complete shape in target domain $\hat { \mathcal { P } } _ { c } ^ { t }$ where GT are not available, we take UCD for guidance. Therefore, the loss function for reconstruction and completion can be expressed by 

$$
\mathcal {L} _ {r e c} ^ {s} = \mathcal {C D} (\mathcal {P} _ {p} ^ {s}, \hat {\mathcal {P}} _ {p} ^ {s}), \mathcal {L} _ {r e c} ^ {t} = \mathcal {C D} (\mathcal {P} _ {p} ^ {t}, \hat {\mathcal {P}} _ {p} ^ {t}),\tag{5}
$$

and 

$$
\mathcal {L} _ {c o m} ^ {s} = \mathcal {C D} (\mathcal {P} _ {c} ^ {s}, \hat {\mathcal {P}} _ {c} ^ {s}), \mathcal {L} _ {c o m} ^ {t} = \mathcal {U C D} (\mathcal {P} _ {p} ^ {t}, \hat {\mathcal {P}} _ {c} ^ {t}),\tag{6}
$$

where $\mathcal { P } _ { p } ^ { s }$ and $\mathcal { P } _ { c } ^ { s }$ are input partial cloud and corresponding complete shape Ground Truth (GT) from source domain respectively, and $\mathcal { P } _ { p } ^ { t }$ indicates the input partial shape from the target domain. 

Domain Factor. To keep the domain-specific features of input partial shapes well preserved in the output prediction, we extract the domain factor to provide domain clues for the decoder. As shown in Fig. 2(a), we utilize a specific disentangler to extract domain factors from common hidden features and introduce a domain discriminator to guide the learning of domain information. Here, our network will predict whether an input partial shape comes from the source domain or target domain according to the domain factor. Then, the domain labels, which are generated automatically, will supervise the domain prediction through cross-entropy loss, guiding the learning of domain factor. 

Domain-Invariant Shape Factor. To make the shape factor domain-invariant, we also utilize the domain discriminator to distinguish the shape factor. However, a gradient reverse layer [11] is utilized between the domain discriminator and shape factor, where gradient will be reversed during back-propagation. Thus, the shape factor can learn to be domain-invariant in an adversarial way. To learn the shape information, this factor will be combined with domain factor to predict the complete point cloud, and the output prediction will be supervised in Eq. (6). 

## 3.2 Factor Permutation Consistency

The independence of each disentangled factor is pursued for an intensive disentanglement [10,26]. To satisfy this property, we introduce a factor permutation consistency loss for the disentanglement of partial point cloud. Specifically, we first feed a batch of B samples into encoder to extract common features, then three separate disentanglers are used to extract occlusion factors $\{ f _ { o } ^ { i } \} _ { i = 1 } ^ { B }$ , domain factors $\{ f _ { d } ^ { i } \} _ { i = 1 } ^ { B }$ and domain-invariant shape features $\{ f _ { s } ^ { i } \} _ { i = 1 } ^ { B }$ respectively. In order to make the shape factors invariant to diferent domain and occlusion situations, we choose to generate random permutations of occlusion features $f _ { o } ^ { i }$ or domain features $f _ { d } ^ { i }$ to form new combinations of factors: 

$$
\tilde {z} ^ {i} = f _ {o} ^ {j} \otimes f _ {d} ^ {i} \otimes f _ {s} ^ {i} \mathrm{or} \tilde {z} ^ {i} = f _ {o} ^ {i} \otimes f _ {d} ^ {j} \otimes f _ {s} ^ {i},\tag{7}
$$

where $j$ is a permutation of i. In our implementation, we attempt to permute occlusion factors or domain factors alternately. As we need to make sure the extracted factors are independent to the remaining factors, an inverse structure of auto-encoder, saying decoder-encoder as shown in Fig. 2(b), is designed to keep factor permutation consistency with the following loss: 

$$
\mathcal {L} _ {c o n s} = \sum_ {i} ^ {B} \| E n c (D e c (\tilde {z} ^ {i})) - \tilde {z} ^ {i} \| _ {2} ^ {2},\tag{8}
$$

where Enc consists of the shared encoder and disentanglers, and $D e c$ indicates the decoder. It is noteworthy that we only add factor permutation consistency loss halfway, when the three factors have been learned preliminarily in encoder. 

## 3.3 Optimization over Disentangled Encoding

Based on the well-trained disentangled representation, we can obtain a complete version of partial point cloud by simply manipulating the occlusion features. To make the overlapping parts between prediction and input partial shape instance look more similar, we introduce a collaboration of regression and optimization method to fine-tune latent factors and decoders within only a few iterations. 

Given the pre-trained auto-encoder from Fig. $2 ( \mathrm { a } ) - ( \mathrm { b } ) ( E n c ^ { \dagger } , D e c ^ { \dagger } )$ and partial point cloud , we first obtain the disentangled factors: 

$$
f _ {o} \otimes f _ {d} \otimes f _ {s} = E n c ^ {\dagger} (\mathcal {P}),\tag{9}
$$

and obtain the initial latent factors of complete shape through: 

$$
z _ {i n i t} = \mathbf {0} \otimes f _ {d} \otimes f _ {s},\tag{10}
$$

as shown in ${ \mathrm { F i g . 2 } } ( \mathrm { c } )$ . Meanwhile, the pre-trained decoder is utilized to initialize the output predictor $D e c _ { i n i t } = D e c ^ { \dagger }$ . Then, we attempt to optimize the disentangled factors and decoder together to better adapt to input partial point clouds by optimizing the following function $\mathcal { L } _ { o p }$ 

$$
z ^ {*}, D e c ^ {*} = \arg \min _ {z, D e c} \mathcal {L} _ {o p} (D e c (z), \mathcal {P}, z),\tag{11}
$$

and the final prediction can be expressed by $D e c ^ { * } ( z ^ { * } )$ 

To construct the loss function, for all points of ${ \mathcal { P } } ,$ we first find their k nearest neighbors in $D e c ( z )$ and the union of all neighboring points form the masked point cloud $M ( D e c ( z ) )$ like ShapeInversion [42]. Then, Chamfer Distance between the partial point cloud and masked complete shape $\mathcal { M C D } ( \mathcal { P } _ { 1 } , \mathcal { P } _ { 2 } ) = \mathcal { C D } ( M ( \mathcal { P } _ { 1 } ) , \mathcal { P } _ { 2 } )$ is used to maximize the similarity. Meanwhile, we also take a regularization of latent factors. All in all, the optimization target is: 

$$
\mathcal {L} _ {o p} (D e c (z), \mathcal {P}, z) = \mathcal {C D} (M (D e c (z)), \mathcal {P}) + \| z \| _ {2} ^ {2}.\tag{12}
$$

Compared with pure optimization method [42], our optimization stage can converge much faster and give much better predictions, since the disentangled factors and well pre-trained decoder have already covered the domain gaps in prediction and give much better initialization for further instance-level adaptation which can be evidenced obviously in Sect. 4.4. 

## 4 Experiments

To show the efectiveness of our method and demonstrate our statement, we treat CRN [32] as our source domain and evaluate the proposed method on the target domain including real-world scans from ScanNet [8], MatterPort3D [4] and KITTI [12] as well as synthesized shape completion dataset 3D-FUTURE [9] and ModelNet [38]. Following previous works [6,42], we assume the category of partial clouds are known in advance and train a separate model for each category. 

## 4.1 Datasets

CRN. We take CRN derived from ShapeNet [5] as our source domain. It provides 30, 174 partial-complete pairs from eight categories where both partial and complete shapes contain 2, 048 points. Here, we take 26, 863 samples from six shared categories between CRN and other datasets for training and evaluation. 

Real-World Scans. Similar to previous works [6,42], we evaluate the performance of our method on partial point cloud from real scans. There are three sources for real scans, saying ScanNet, MatterPort3D, and KITTI. The tables and chairs in ScanNet and MatterPort3D, and cars in KITTI are used for performance evaluation. We re-sample the input scans to 2, 048 points for unpaired training and inference to match the virtual dataset. 

3D-FUTURE. To evaluate the performance on more realistic shapes, we generate another point cloud completion dataset from 3D-FUTURE [9]. The models in 3D-FUTURE are much more close to real objects. Similarly, we obtain partial shapes and complete ones with 2, 048 points from 5 diferent view-points. Because 3D-FUTURE only contains indoor furniture models, we only take five shared categories of furniture for point cloud completion. 

ModelNet. We generate a shape completion dataset ModelNet using models from ModelNet40 [38]. We synthesize the partial shape through virtually scanning and generate complete ones by randomly sampling points in the surface like previous works [31,41]. 2, 048 points are taken for both partial and complete shapes to match the CRN dataset. In order to test the adaptation ability, we take the shared categories of ModelNet40 and CRN for evaluation. 

## 4.2 Implementation

All experiments can be conducted on a machine with GTX 1080Ti and 64 GB RAM. Here, we take PointNet [25] as our encoder to extract common features with dimension 1, 024. Then, we take three separate disentanglers consisting of two MLPs to extract $f _ { o } \in \mathbb { R } ^ { 9 6 } , f _ { d } \in \mathbb { R } ^ { 9 6 }$ , and $f _ { s } \in \mathbb { R } ^ { 9 6 }$ , and a TreeGCN [27] as our decoder. More details is available at https://github.com/azuki-miho/ OptDE. 

## 4.3 Metrics and Results

Metrics. For real scans without ground truth, we use Unidirectional Chamfer Distance (UCD) and Unidirectional Hausdorf Distance (UHD) from the partial input to the predicted complete shapes as our metric following previous works [6, 36,42]. For more comprehensive evaluation of our completion performance on cross-domain datasets, we take mean Chamfer Distance as our metric for the brand new datasets 3D-FUTURE and ModelNet like previous works [30,41] where complete shapes are available for testing. 

Here, we first compare our method with the prevailing unsupervised crossdomain completion methods on real-world datasets of ScanNet, MatterPort3D and KITTI. The results are reported in Table 1 where UCD and UHD are used as metrics for evaluation. In this table, DE indicates regression method only using disentangled encoding shown in Fig. 2(a)–(b), and OptDE shows the results of optimization over disentangled encoding (Fig. 2(c)). As shown, disentangled encoding significantly improves the completion performance on real-world scans, and optimization over the disentangled encoding can further refine the results according to the input partial shapes. That is because our method can cover the domain gaps in output prediction between diferent datasets and adapt to various instances even within the target domain. We also show the qualitative results in Fig. 3 where our predictions correspond well to input partial scans. 


Table 1. Cross-domain completion results on real scans. We take [UCD /UHD ] as our metrics to evaluate the performance, and the scale factors are $1 0 ^ { 4 }$ for UCD and $1 0 ^ { 2 }$ for UHD. +UHD indicates UHD loss is used during training.


<table><tr><td rowspan="2">Methods</td><td colspan="2">ScanNet</td><td colspan="2">MatterPort3D</td><td>KITTI</td></tr><tr><td>Chair</td><td>Table</td><td>Chair</td><td>Table</td><td>Car</td></tr><tr><td>pcl2pcl [6]</td><td>17.3/10.1</td><td>9.1/11.8</td><td>15.9/10.5</td><td>6.0/11.8</td><td>9.2/14.1</td></tr><tr><td>ShapeInversion [42]</td><td>3.2/10.1</td><td>3.3/11.9</td><td>3.6/10.0</td><td>3.1/11.8</td><td>2.9/13.8</td></tr><tr><td>+UHD [42]</td><td>4.0/9.3</td><td>6.6/11.0</td><td>4.5/9.5</td><td>5.7/10.7</td><td>5.3/12.5</td></tr><tr><td>Cycle4Compl. [33]</td><td>5.1/6.4</td><td>3.6/5.9</td><td>8.0/8.4</td><td>4.2/6.8</td><td>3.3/5.8</td></tr><tr><td>DE (Ours)</td><td>2.8/5.4</td><td>2.5/5.2</td><td>3.8/6.1</td><td>2.5/5.4</td><td>1.8/3.5</td></tr><tr><td>OptDE (Ours)</td><td>2.6/5.5</td><td>1.9/4.6</td><td>3.0/5.5</td><td>1.9/5.3</td><td>1.6/3.5</td></tr></table>

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/b4ba5f1fffeab4a8706ed9e1a4c132990d5d3a8c3735f128a484f077a0032865.jpg)



Fig. 3. Visualization results on the data of ScanNet, MatterPort3D and KITTI. Partial point clouds, predictions of pcl2pcl, ShapeInversion, Cycle4Completion and our methods are presented separately from the left to the right.


Additionally, we report the completion results of our method and previous works on target domain 3D-FUTURE in Table 2, and only complete shapes of CRN and partial point clouds of 3D-FUTURE are used for training for fair comparison. In this dataset, our method significantly outperforms other competitors. Again, collaboration of regression and optimization can improve the performance by adapting to each instance. Figure 4(a) gives the visualization results of our method and shows the qualitative improvement over previous works. 


Table 2. Results of cross-domain completion on 3D-FUTURE. We evaluate the performance of each method using [CD ] and scale-up factor is 10<sup>4</sup>.


<table><tr><td>Methods</td><td>Cabinet</td><td>Chair</td><td>Lamp</td><td>Sofa</td><td>Table</td><td>Avg.</td></tr><tr><td>Pcl2pcl [6]</td><td>57.23</td><td>43.91</td><td>157.86</td><td>63.23</td><td>141.92</td><td>92.83</td></tr><tr><td>ShapeInversion [42]</td><td>38.54</td><td>26.30</td><td>48.57</td><td>44.02</td><td>108.60</td><td>53.21</td></tr><tr><td>Cycle4Compl. [33]</td><td>32.62</td><td>34.08</td><td>77.19</td><td>43.05</td><td>40.00</td><td>45.39</td></tr><tr><td>DE (Ours)</td><td>28.62</td><td>22.18</td><td>30.85</td><td>38.01</td><td>27.43</td><td>29.42</td></tr><tr><td>OptDE (Ours)</td><td>28.37</td><td>21.87</td><td>29.92</td><td>37.98</td><td>26.81</td><td>28.99</td></tr></table>


Table 3. Results of cross-domain completion on ModelNet. We take [CD ] as our metric to evaluate the performance of each method which has been scaled by 10<sup>4</sup>.


<table><tr><td>Methods</td><td>Plane</td><td>Car</td><td>Chair</td><td>Lamp</td><td>Sofa</td><td>Table</td><td>Avg.</td></tr><tr><td>Pcl2pcl [6]</td><td>18.53</td><td>17.54</td><td>43.58</td><td>126.80</td><td>38.78</td><td>163.62</td><td>68.14</td></tr><tr><td>ShapeInversion [42]</td><td>3.78</td><td>15.66</td><td>22.25</td><td>60.42</td><td>22.25</td><td>125.31</td><td>41.61</td></tr><tr><td>Cycle4Compl. [33]</td><td>5.77</td><td>11.85</td><td>26.67</td><td>83.34</td><td>22.82</td><td>21.47</td><td>28.65</td></tr><tr><td>DE (Ours)</td><td>2.19</td><td>9.80</td><td>15.11</td><td>42.94</td><td>21.45</td><td>10.26</td><td>16.96</td></tr><tr><td>OptDE (Ours)</td><td>2.18</td><td>9.80</td><td>14.71</td><td>39.74</td><td>19.43</td><td>9.75</td><td>15.94</td></tr></table>

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/35722f10d9d8d59d66b53ae0e980242d9a2336cc83559913dc017d5a2fe147b9.jpg)



Fig. 4. Visualization results on the test set of 3D-FUTURE and ModelNet. The images from the top to bottom are input partial clouds, results given by pcl2pcl, ShapeInversion, Cycle4Completion and ours, and Ground Truth respectively.


We further compare the cross-domain completion performance on target domain dataset ModelNet. The results of cross-domain completion on this dataset are reported in Table 3 where disentangled encoding alone can outperform previous methods by a large margin. In addition, optimization over the disentangled encoding can further boost the performance especially for hard categories. 

Moreover, we provide the qualitative results of diferent methods in Fig. 4(b). As can be seen, our method can well adapt to input partial shapes from diferent domains. 

## 4.4 Ablation Study

In this section, we will conduct more experiments to evaluate the efectiveness of our proposed method from diferent aspects and prove our claims. Without loss of generality, we mainly utilize CRN as the source domain and evaluate on the target domain ModelNet. 

Optimization over Disentangled Encoding. In order to evaluate the efectiveness of diferent parts in our method and test how far away from a perfect crossdomain completion method, we conduct ablation studies as follows. We first train the network with the same structure using only paired point clouds from CRN and evaluate the performance on ModelNet which is taken as our baseline. Then, we add Disentangled Representation Learning (Fig. 2(a)), Factor Permutation Consistency, and Optimization stage gradually. Additionally, we evaluate the best performance that can be brought by our backbones through training using paired data from both source domain CRN and target domain ModelNet, which is usually named as the oracle. We report all the results in Table 4. 


Table 4. Ablation study of occlusion factor supervision on ModelNet. $\mathrm { [ C D \downarrow ] ( \times 1 0 ^ { 4 } ) }$ is taken as our metric to evaluate the performance improvement and distance to oracle.


<table><tr><td>Methods</td><td>Plane</td><td>Car</td><td>Chair</td><td>Lamp</td><td>Sofa</td><td>Table</td><td>Avg.</td></tr><tr><td>Baseline</td><td>5.41</td><td>10.05</td><td>22.82</td><td>67.25</td><td>22.44</td><td>53.14</td><td>30.19</td></tr><tr><td>DE w/o Consistency</td><td>2.27</td><td>10.05</td><td>15.36</td><td>46.18</td><td>22.08</td><td>11.09</td><td>17.84</td></tr><tr><td>+ Consistency</td><td>2.19</td><td>9.80</td><td>15.11</td><td>42.94</td><td>21.45</td><td>10.26</td><td>16.96</td></tr><tr><td>+ Optimization</td><td>2.18</td><td>9.80</td><td>14.71</td><td>39.74</td><td>19.43</td><td>9.75</td><td>15.94</td></tr><tr><td>Oracle</td><td>1.51</td><td>6.58</td><td>10.52</td><td>41.98</td><td>9.94</td><td>7.87</td><td>13.07</td></tr></table>

It shows that our method can greatly handle the domain gaps in the output space and well preserve domain-specific patterns in predictions thanks to the disentanglement of occlusion factor and domain factor. Permutation consistency loss and optimization over the disentangled representation can both boost the performance. Even though, the improvement on car and sofa category is minor and that is because the samples in source domain have covered most samples in the target domains but the distribution is quite diferent. Compared with the oracle, there are still gaps to be bridged. Thus, this paper may inspire more work to focus on how to transfer the knowledge of virtual shapes to real objects given only virtual complete shapes and real partial scans. 

Occlusion Factor Manipulation. In order to show the learning of disentangled occlusion factor and prove our claims, we take four original partial point clouds $\{ \mathcal { P } _ { i } \} _ { i = 1 } ^ { 4 }$ that are scanned from diferent view-points, and then utilize the shared encoder and disentanglers to obtain the occlusion factors, domain factors and domain-invariant shape factors. After that, we replace the occlusion factors of $\mathcal { P } _ { 1 }$ and $\mathcal { P } _ { 3 }$ by those of $\mathcal { P } _ { 2 }$ and $\mathcal { P } _ { 4 }$ , and obtain new generated point clouds through the decoder as shown in Fig. 5. 

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/18e8c942bf093e594a549db03c53f328182da48521fff44759f888971fb46127.jpg)



Fig. 5. Visualization of occlusion factor manipulation. The disentangled occlusion factors of $\mathcal { P } _ { 1 }$ and $\mathcal { P } _ { 3 }$ are replaced by the occlusion factors of $\mathcal { P } _ { 2 }$ and $\mathcal { P } _ { 4 }$ . The new latent factors can generate brand-new partial point clouds through the pre-trained decoder. (Color figure online)


We can see the back and seat of $\mathcal { P } _ { 1 }$ is occluded (blue circle), and the right front leg (red circle) of $\mathcal { P } _ { 2 }$ is occluded. After replacing the occlusion factor, the right front leg of $\mathcal { P } _ { 1 } ^ { g }$ is occluded due to the occlusion factor while shape and domain information is well preserved. We can also see the occlusion factor manipulation efect in $\mathcal { P } _ { 2 } ^ { g }$ . This indicates a disentangled representation can provide a much easier way to control the occlusion through simple factor manipulation. 

Initialization in Optimization. Our method pursue a collaboration of regression and optimization where disentangled encoding provides a good initialization for optimization. Here, we provide two representative examples of optimization progression in Fig. 6 where our method can provide a desirable prediction, and the optimization can converge within about 4 iterations thanks to this good initialization of latent code and decoder. Compared with ours, ShapeInversion converges much slower and may even converge to a sub-optimal solution. 

![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/71dfa485e29fd0c4ef50ee1763a09a35b92e2a0c445229cb4ca355381702596c.jpg)


![image](https://cdn-mineru.openxlab.org.cn/result/2026-08-24/bd35acb5-e73e-44d1-bc33-2c438269be59/c37d16d34df411bd501878ab41443d07ac0636c8def5d1b8c71f3f956a4e31ec.jpg)



Fig. 6. Optimization progression on ModelNet test set. Compared with ShapeInversion, our method can converge 100 faster (0.12 s for 4 iterations v.s. 23.56 s for 800 iterations on a single GTX 1080Ti) and easily circumvent sub-optimal solution.


## 5 Conclusion

In this paper, we propose the very first method OptDE to deal with the output domain gap in shape completion. We introduce a disentangled representation consisting of three essential factors for any partial shape, and shape completion can be implemented by simply manipulating the occlusion factor while preserving shape and domain features. To further adapt to each partial instance in the target domain, we introduce a collaboration of regression and optimization to ensure the consistency between completed shapes and input scans. For comprehensive evaluation on cross-domain completion, we treat CRN as the source domain and evaluate on real-world scans in ScanNet, MatterPort3D and KITTI as well as synthesized datasets 3D-FUTURE and ModelNet. Results show that our method outperforms previous methods by a large margin which may inspire more works to focus on cross-domain point cloud completion. 

Limitation and Discussion. Since all previous methods assume the category of partial shapes to be known and trained in category-specific way, we believe it will be better to train a unified model for cross-domain completion of all categories. 

Acknowledgments. This work is sponsored by the National Key Research and Development Program of China (No. 2019YFC1521104), the National Natural Science Foundation of China (No. 61972157,72192821), Shanghai Municipal Science and Technology Major Project (2021SHZDZX0102), Shanghai Sailing Program (22YF1420300), Shanghai Science and Technology Commission (21511101200) and SenseTime Collaborative Research Grant. 

## References



1. Aberman, K., Li, P., Lischinski, D., Sorkine-Hornung, O., Cohen-Or, D., Chen, B.: Skeleton-aware networks for deep motion retargeting. ACM Trans. Graph. (TOG) 39(4), 62-1 (2020) 





2. Barlow, H.B., Kaushal, T.P., Mitchison, G.J.: Finding minimum entropy codes. Neural Comput. 1(3), 412–423 (1989) 





3. Bau, D., et al.: Semantic photo manipulation with a generative image prior. In: SIGGRAPH (2020) 





4. Chang, A., et al.: Matterport3D: learning from RGB-D data in indoor environments. In: 2017 International Conference on 3D Vision (3DV), pp. 667–676. IEEE Computer Society (2017) 





5. Chang, A.X., et al.: ShapeNet: an information-rich 3d model repository. arXiv preprint arXiv:1512.03012 (2015) 





6. Chen, X., Chen, B., Mitra, N.J.: Unpaired point cloud completion on real scans using adversarial training. In: International Conference on Learning Representations (2020) 





7. Cosmo, L., Norelli, A., Halimi, O., Kimmel, R., Rodol`a, E.: LIMP: learning latent shape representations with metric preservation priors. In: Vedaldi, A., Bischof, H., Brox, T., Frahm, J.-M. (eds.) ECCV 2020. LNCS, vol. 12348, pp. 19–35. Springer, Cham (2020). https://doi.org/10.1007/978-3-030-58580-8 2 





8. Dai, A., Chang, A.X., Savva, M., Halber, M., Funkhouser, T., Nießner, M.: Scan-Net: richly-annotated 3d reconstructions of indoor scenes. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 5828–5839 (2017) 





9. Fu, H., et al.: 3d-future: 3d furniture shape with texture. arXiv preprint arXiv:2009.09633 (2020) 





10. Fumero, M., Cosmo, L., Melzi, S., Rodol`a, E.: Learning disentangled representations via product manifold projection. In: ICML (2021) 





11. Ganin, Y., Lempitsky, V.: Unsupervised domain adaptation by backpropagation. In: International Conference on Machine Learning, pp. 1180–1189. PMLR (2015) 





12. Geiger, A., Lenz, P., Urtasun, R.: Are we ready for autonomous driving? The Kitti vision benchmark suite. In: 2012 IEEE Conference on Computer Vision and Pattern Recognition, pp. 3354–3361. IEEE (2012) 





13. Gonzalez-Garcia, A., van de Weijer, J., Bengio, Y.: Image-to-image translation for cross-domain disentanglement. In: NeurIPS (2018) 





14. Hou, J., Dai, A., Nießner, M.: RevealNet: seeing behind objects in RGB-D scans. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 2098–2107 (2020) 





15. Huang, Z., Yu, Y., Xu, J., Ni, F., Le, X.: PF-Net: point fractal network for 3d point cloud completion. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 7662–7670 (2020) 





16. Kim, H., Mnih, A.: Disentangling by factorising. In: International Conference on Machine Learning (ICML), pp. 2649–2658. PMLR (2018) 





17. Kingma, D.P., Welling, M.: Auto-encoding variational bayes. In: ICLR (2014) 





18. Kolotouros, N., Pavlakos, G., Black, M.J., Daniilidis, K.: Learning to reconstruct 3d human pose and shape via model-fitting in the loop. In: Proceedings of the IEEE/CVF International Conference on Computer Vision, pp. 2252–2261 (2019) 





19. Liu, A.H., Liu, Y.C., Yeh, Y.Y., Wang, Y.C.F.: A unified feature disentangler for multi-domain image translation and manipulation. In: Proceedings of the 32nd International Conference on Neural Information Processing Systems, pp. 2595– 2604 (2018) 





20. Liu, M., Sheng, L., Yang, S., Shao, J., Hu, S.M.: Morphing and sampling network for dense point cloud completion. In: Proceedings of the AAAI Conference on Artificial Intelligence, vol. 34, pp. 11596–11603 (2020) 





21. Liu, Y.C., Yeh, Y.Y., Fu, T.C., Wang, S.D., Chiu, W.C., Wang, Y.C.F.: Detach and adapt: learning cross-domain disentangled deep representation. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 8867–8876 (2018) 





22. Ma, F., Ayaz, U., Karaman, S.: Invertibility of convolutional generative networks from partial measurements. In: Advances in Neural Information Processing Systems, vol. 31 (2018) 





23. Pan, L., et al.: Variational relational point completion network. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 8524–8533 (2021) 





24. Peng, X., Huang, Z., Sun, X., Saenko, K.: Domain agnostic learning with disentangled representations. In: International Conference on Machine Learning, pp. 5102–5112. PMLR (2019) 





25. Qi, C.R., Su, H., Mo, K., Guibas, L.J.: PointNet: deep learning on point sets for 3d classification and segmentation. In: The IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), pp. 652–660 (2017) 





26. Schmidhuber, J.: Learning factorial codes by predictability minimization. Neural Comput. 4(6), 863–879 (1992) 





27. Shu, D.W., Park, S.W., Kwon, J.: 3d point cloud generative adversarial network based on tree structured graph convolutions. In: Proceedings of the IEEE/CVF International Conference on Computer Vision, pp. 3859–3868 (2019) 





28. Sigal, L., Balan, A., Black, M.: Combined discriminative and generative articulated pose and non-rigid shape estimation. Adv. Neural. Inf. Process. Syst. 20, 1337– 1344 (2007) 





29. Straßer, W.: Schnelle kurven-und fl¨achendarstellung auf grafischen sichtger¨aten. Ph.D. thesis (1974) 





30. Tchapmi, L.P., Kosaraju, V., Rezatofighi, H., Reid, I., Savarese, S.: TopNet: structural point cloud decoder. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 383–392 (2019) 





31. Wang, H., Liu, Q., Yue, X., Lasenby, J., Kusner, M.J.: Unsupervised point cloud pre-training via occlusion completion. In: Proceedings of the IEEE/CVF International Conference on Computer Vision, pp. 9782–9792 (2021) 





32. Wang, X., Ang Jr, M.H., Lee, G.H.: Cascaded refinement network for point cloud completion. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 790–799 (2020) 





33. Wen, X., Han, Z., Cao, Y.P., Wan, P., Zheng, W., Liu, Y.S.: Cycle4completion: unpaired point cloud completion using cycle transformation with missing region coding. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 13080–13089 (2021) 





34. Wen, X., Li, T., Han, Z., Liu, Y.S.: Point cloud completion by skip-attention network with hierarchical folding. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 1939–1948 (2020) 





35. Wen, X., et al.: PMP-Net: point cloud completion by learning multi-step point moving paths. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 7443–7452 (2021) 





36. Wu, R., Chen, X., Zhuang, Y., Chen, B.: Multimodal shape completion via conditional generative adversarial networks. In: Vedaldi, A., Bischof, H., Brox, T., Frahm, J.-M. (eds.) ECCV 2020. LNCS, vol. 12349, pp. 281–296. Springer, Cham (2020). https://doi.org/10.1007/978-3-030-58548-8 17 





37. Wu, X., Huang, H., Patel, V.M., He, R., Sun, Z.: Disentangled variational representation for heterogeneous face recognition. In: Proceedings of the AAAI Conference on Artificial Intelligence, vol. 33, pp. 9005–9012 (2019) 





38. Wu, Z., et al.: 3d shapenets: a deep representation for volumetric shapes. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 1912–1920 (2015) 





39. Xie, H., Yao, H., Zhou, S., Mao, J., Zhang, S., Sun, W.: GRNet: gridding residual network for dense point cloud completion. In: Vedaldi, A., Bischof, H., Brox, T., Frahm, J.-M. (eds.) ECCV 2020. LNCS, vol. 12354, pp. 365–381. Springer, Cham (2020). https://doi.org/10.1007/978-3-030-58545-7 21 





40. Yang, Y., Feng, C., Shen, Y., Tian, D.: FoldingNet: point cloud auto-encoder via deep grid deformation. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 206–215 (2018) 





41. Yuan, W., Khot, T., Held, D., Mertz, C., Hebert, M.: PCN: point completion network. In: 2018 International Conference on 3D Vision (3DV), pp. 728–737. IEEE (2018) 





42. Zhang, J., et al.: Unsupervised 3d shape completion through GAN inversion. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 1768–1777 (2021) 





43. Zhang, W., Yan, Q., Xiao, C.: Detail preserved point cloud completion via separated feature aggregation. In: Vedaldi, A., Bischof, H., Brox, T., Frahm, J.-M. (eds.) ECCV 2020. LNCS, vol. 12370, pp. 512–528. Springer, Cham (2020). https://doi. org/10.1007/978-3-030-58595-2 31 

