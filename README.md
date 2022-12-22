# Rover Segmentation Using Kmeans

## Introduction

In this project, the goal is to segment the picture into rover, background, and shadow (if any). The whole algorithm is illustrated below in Figure 1. Each section is discussed respectively and the final result is shown in table 1.

![image](https://user-images.githubusercontent.com/51737180/209217791-2b1de361-89e5-49ad-a537-0241a5c14823.png)

Figure 1 The illustration of the algorithm

## Method & Results

#### Kmeans

Kmeans is one of the clustering algorithms that aims to cluster data based on their distance to the centroid (cluster centers). The only input parameter is the number of clusters (k) although we can try different distance functions and initial states. Since the initial state is so important to not get stuck in a local minimum, for each fitting we set the number of iterations to 10. We implemented two distance functions including Euclidean distance and cosine. The results with cosine were not so promising compared to the Euclidean function and besides that, it takes a lot of time to fit it. The reason might be that the computation cost of dot product over magnitude for every two points is too much, so we were not able to analyze this distance function comprehensively. Nevertheless, a few times of trials showed that the Euclidean distance function may outperform cosine.

Moreover, to find the best number of clusters we tried GAP statistics to determine the number of clusters. Although it is stated[^1] that when clusters are similar together the GAP[^2] method is likely to overestimate the K, we found that by increasing the number of clusters the model can better classify the shadow in picture 4. The GAP value is shown in Figure 2. For the most combination of settings, the Gap plot was the same as Figure 2, which we found better at discriminating the shadow in picture 4. Therefore, all results are based on K=14.

[^1]: https://towardsdatascience.com/k-means-clustering-and-the-gap-statistics-4c5d414acd29
[^2]: Tibshirani, Walther, and Hastie, "Estimating the Number of Clusters in a Data Set via the Gap Statistic."

![image](https://user-images.githubusercontent.com/51737180/209216209-4eaff6e8-37a5-4dce-8975-0051bf6795f7.png)

Figure 2 GAP values


Table 1 Final results of segmentation with different combination of settings (The white area is the segmented part in each image)
![image](https://user-images.githubusercontent.com/51737180/209219965-536328b2-5c80-4f3a-b2fc-f1fd89073fb7.png)
![image](https://user-images.githubusercontent.com/51737180/209220015-5626aa45-89c9-489f-b7d0-50e9ebaf14e0.png)


#### Color space

First, we tried three-color spaces including RGB, HSI, and HSV. Since the input of the K-means algorithm is the number of samples * the number of features, each channel of the pixel value is one feature of the input data. What we expected is that HSI and HSV should be similar due to a small difference between them. In HSI maximum intensity is pure white while in HSV maximum value is a color with a white light shining at that. As it is shown in rows 1, 5, and 9 the overall quality of segmentation is the same but in RGB space we can see a better result (for example picture 3).

#### Decorrelation stretching 

Since the value of each channel is close to each other, decorrelation stretching may help the model to discriminate the different areas of the image more easily. In Figure 3 the 3D scatter plot of the original image (picture 2) is shown. To further illustrate the pixel values in the RGB space, a 2D plot of the image before and after decorrelation stretch is shown in Figures 4 and 5 respectively. As it illustrates, the difference between clusters (rover and background) is not apparent since it is not separated. Therefore, it is expected to not have a good result in a clustering of picture 2. Nevertheless, we tried decorrelation stretching and finally found out that it not only does not help the segmentation but also it worsens the result such as in picture 4, row 1, and row 3 of table 1. This is because by expanding the points, the K-means algorithm is likely to cluster an individual almost uniform area into two separate clusters. Hence, in most cases using decorrelation stretching does not help the operation. 

![image](https://user-images.githubusercontent.com/51737180/209218427-e0540e69-643f-4a5b-855c-a3e77adec683.png)

Figure 3 3D scatter plot of the original image (left) and image after decorrelation stretch (right)

![image](https://user-images.githubusercontent.com/51737180/209218483-1a44d2e5-b408-41d8-82d2-9b74a1c8b482.png)

Figure 4 2D scatter plot of the Green and red channels of the original image (left) and image after decorrelation stretch (right)

![image](https://user-images.githubusercontent.com/51737180/209218563-bf65a176-f57c-4aa2-89e2-a4ba8547b0ba.png)

Figure 5 2D scatter plot of the blue and green channels of the original image (left) and image after decorrelation stretch (right)

#### Position as a feature 

In some cases that there is a similarity in color information of the different areas of the image, the K-means algorithm may mistakenly get these points as a cluster. Consequently, we can stack the location data of each pixel to the feature vector and increase the number of features to five. In this case, it is expected that the K-means algorithm can consider the color as well as the location, so it is more likely to cluster a group of pixels close together as a cluster. As shown in Table 1, this expectation is true in most cases. For instance, segmenting shadow with the help of position info in picture 4 (in rows 9 and 10 of Table 1) is slightly better than without position information. This is also true for pictures 2 and 3. It is worth noting that when we add the position feature and do the correlation stretching simultaneously, the results are no longer enhanced. This could be because when we implement correlation stretching, the variance between color intensities increases a lot so that the K-means algorithm is more likely to discriminate the data based on pixel values (intensities) less on the location of the pixel. 

#### Morphological Operations 

After merging the masks that belonged to one cluster, we implemented morphological operations on images. First, to remove the islands we utilized a kind of erosion that removes small objects by setting a pixel to 0 if all certain number of neighbors are 1. In the SciPy package, it is called “binary_filled_holes”. After that, by applying morphological opening operation. This operation erodes and dilates the image back-to-back so that the shape and size of the image are preserved but removes small objects in the mask. This is shown in Figure 6. 

![image](https://user-images.githubusercontent.com/51737180/209218777-7f05b984-47cb-4632-8bf3-7076552ec54b.png)

Figure 6  The mask of picture 2 second row of table 1. The left one is before applying morphological operations (erosion and opening) and the right one is after mentioned operations 

The final result after applying morphological operations for picture 2 is shown in Figure 7. 

![image](https://user-images.githubusercontent.com/51737180/209218860-9b84c02c-2bf4-418b-976a-caedda87736b.png)

Figure 7  Rover segmented in picture 2 

## Conclusion 

To conclude, the best result is obtained by using the HSV color space plus adding position as a feature. This is because the third value of the image in HSV contains the color information which is a more discriminative feature compared to the RGB channels. Plus, by adding the location of each pixel to this, the result is going to be better. In Figure 8, picture 4 is shown in RGB and HSV color space. 

![image](https://user-images.githubusercontent.com/51737180/209219070-1a6ad6ef-aefc-4d3f-ab44-129c07dee73e.png)

Figure 8  Picture 4 in RGB and HSV color space 

To sum up, the best results are obtained in HSV color space and by adding the position feature. Moreover, the better distance function for Kmeans is the Euclidean distance function compared to cosine. The segmented part (rover) of pictures 1, 3, and 4 are shown in Figures 9, 10, and 11 respectively. 

![image](https://user-images.githubusercontent.com/51737180/209219153-c63a50a8-4d94-4978-943d-173b8ae9c7f1.png)

Figure 9  Segmented Rover in picture 1 

![image](https://user-images.githubusercontent.com/51737180/209219208-8bf1e1ce-f6f2-44cf-b6ae-b8ac49e2add6.png)

Figure 10  Segmented rover in picture 3 

![image](https://user-images.githubusercontent.com/51737180/209219241-bd4c45f8-939d-48dc-b4db-e14cd1e1524e.png)

Figure 11  Segmented rover in picture 4 

![image](https://user-images.githubusercontent.com/51737180/209219266-871a2db1-3978-4c0f-833e-b487c75db43a.png)

Figure 12  Segmented shadow in picture 4 

