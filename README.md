# Customer Segmentation Using K-Means Algorithm

## Abstract
The following project uses the [RFM](https://en.wikipedia.org/wiki/RFM_(market_research)) method in order to segment customers of a UK based retail company, based on the 'Recency', 'Frequency' and 'Monetary Value' of their transactions. The data used can be found on the [UCI ML site](https://archive.ics.uci.edu/dataset/502/online+retail+ii), and spans years 2010 - 2011. The algorithm used to uncover clustering within the data was [K-Means](https://en.wikipedia.org/wiki/K-means_clustering), which consists of an iterative process with a assignment phase and update phase at each step. Within the assignment phase, each datapoint is assigned to its closest centroid (cluster centers of which K exist). In the update phase the position of each centroid is recomputed relative to its assigned datapoints, by taking their combined mean value. K-Means was an optimal algorithmic choice for this task as it is fast and simple, with clear cluster interpretability. 

## I. Exploratory Data Analysis and Data Cleaning
The initial analysis of the data revealed several inconsistencies. For instance, the 'quantity' and 'price' columns contained negative values, which were dropped prior to conducting the analysis. Regex patten matching was also used to ensure that invoice and stock records aligned with the format specified by the metadata, and instances that failed to conform to this structure were also dropped. After preprocessing, about 77% of the original data was retained. 

## II. Feature Engineering
In order to conduct RFM segmentation, the individual features of recency, frequency and monetary value had to be engineered. This was achieved by calculating the difference between each customers last invoice date and the EOY date, counting the number of unique invoices for each customer within the period and calculating the sum of all invoices for each customer respectively. After engineering these features, histograms of each feature were drawn in order to determine the distribution of the feature variables and to identify potential outliers. Drawing box plots for each feature variable further confirmed that the Monetary Value and Frequency distributions were distorted by a few extreme values. These were subsequently separated for isolated analysis, in order to prevent them disrupting the clustering algorithm (which is sensitive to such outliers). 

<img width="1556" height="525" alt="output" src="https://github.com/user-attachments/assets/83b5c379-18ac-4aed-9137-791303643ca7" />
<img width="1890" height="590" alt="Boxplots" src="https://github.com/user-attachments/assets/c4e91096-a4bc-4dd0-998d-ab5ef1cc0cbf" />

Removal of outliers resulted in a much more even distribution of data : 

<img width="1890" height="590" alt="output2" src="https://github.com/user-attachments/assets/cafb2872-c591-46de-8709-a858aa963534" />

## III. K-Means Clustering
To visualize the three feature variables, a 3-D scatter plot was generated. Prior to the K-Means algorithm, the feature variables were also scaled to allow for more effective clustering. 

<img width="645" height="636" alt="initialscatter" src="https://github.com/user-attachments/assets/b43de519-4a65-4efe-a5ae-111f1a9134d7" />

Though K-Means has several advantages in that it is quick (with O(n·k·d·i) time complexity where n, k, d, i = no. datapoints, no. clusters, no. dimensions, no. iterations), and simple to interpret, its drawback is that the number of clusters is predetermined and thus must be set beforehand. Thus the algorithm must be repeated several time across a variety of cluster initializations in order to determine the optimal cluster number. After doing this, the inertia values for each initialization were obtained. These represent the average sum of squared distances between the centroid of each cluster and its assigned points, with a lower score evidencing better clustering. These values are used to draw what is termed an 'elbow plot'.

<img width="994" height="659" alt="elbow" src="https://github.com/user-attachments/assets/8a071597-e920-40b6-b46e-2622961bd6cb" />

The inertia will decrease naturally with the number of clusters as k tends towards the number of datapoints. With more centroids available, datapoints will naturally sit closer to one of these cluster centers. Thus selecting the number of clusters purely with the objective of minimizing inertia is an ineffective strategy. Instead, the optimal practice is to select the number of clusters which correspond to the 'elbow' of the elbow plot, i.e. the inflection point which in the above diagram sits subjectively at either k = 3 or k = 4. 

In order to eliminate this subjectivity the [silhouette scores](https://en.wikipedia.org/wiki/Silhouette_(clustering)) for each sample were used to more robustly determine the optimal number of clusters. The silhouette score measures how well each point fits its own cluster compared to the nearest other cluster and ranges from -1 to 1. A score closer to 1 indicates that a point more strongly belongs ton its assigned cluster, whilst a score closer to -1 suggests a point may be wrongly assigned. Roughly, the score is computed by comparing the average distance of a sample to the other points in its own cluster and the average distance of that same sample to the points in the nearest neighboring cluster. From these derived silhouette scores, a silhouette diagram can be plotted (also termed a 'knife' diagram). The dashed red line corresponds to the average silhouette score across all clusters. The knife plots show the distribution of silhouette scores for each cluster. The width of each knife plot corresponds to the number of points assigned to that cluster. 

<img width="1490" height="1009" alt="knife" src="https://github.com/user-attachments/assets/8765f3fa-b0d3-4946-990c-1f86e7a36dee" />

Comparing the silhouette plots of clusters 3 and 4, we observe that k = 3 has a higher average silhouette score (0.458 vs. 0.415). A naive strategy would thus favor 3 clusters over 4. However observing the shape of the knife plots, we can see that k = 4 evidences more balanced clustering, and thus on that basis we select 4 as our number of clusters for the K-Means algorithm. 

The algorithm was then run and the data was re-plotted with a color scheme mapping each point to its corresponding label. 

<img width="636" height="636" alt="clusterscatter" src="https://github.com/user-attachments/assets/ce95c791-c6b5-463a-84f8-968eaf869592" />

## IV. Cluster Analysis
To further gain insight into the distributions of each cluster, violin plots were generated to allow for subsequent analysis. The gray plots show the distribution across all customers for the feature in question.
<img width="1479" height="833" alt="violin1" src="https://github.com/user-attachments/assets/c36798c0-4f55-4b81-af66-93ea99fa01be" />

### Cluster 1 : 
These customers show low recency, frequency and monetary value. These customers represent buyers who are not regular customers and don't have any particular brand loyalty to the company. Their purchases are sparse and perhaps for a few items which they may have demanded within the year for which the company was able to provide supply for. The strategy for these customers is to engage with them in order to increase their brand loyalty and convert them into regular customers. Possible actions would be to increase advertising targeted towards this particular group.

### Cluster 2 : 
These customers evidence the highest monetary value and frequency, and thus are most valuable to the company, however they display low recency. A potential strategy could be to re-engage these customers via targeted advertising and small incentives such as discounts. Discounting will not be needed much as these customers are already high spenders.

### Cluster 3 : 
These customers have the most recent activity, however their spending amount and frequency is low. The strategy should be to increase these variables perhaps by offering bundle deals.

### Cluster 4: 
Customers in this cluster show above average spending and frequency though these characteristics are not exceptionally high. Recency for these customers also is shown to be average. These customers are therefore more representative of the regular consumer of this company's goods, and a potential strategy could be to incentivize increased activity by offering discounts on certain select high list items or bundles. 

A summary of the information derived from cluster analysis is presented below.

<img width="1050" height="679" alt="final" src="https://github.com/user-attachments/assets/81920476-7523-4bed-be1b-670e2499ca3b" />

Outlier customers were analyzed separately and formed three distinct groups : Monetary Outliers (M), Frequency Outliers (F) and Monetary and Frequency Outliers (M&F). 

### Cluster M : 
Customers have abnormally high monetary spending but lack frequency of purchase. One solution could be loyalty programs such as monthly subscription memberships in order to increase their activity.

### Cluster F : 
Customers have very high frequency but lower monetary value. This evidences strong brand loyalty but also product selection. A potential solution is to offer these customers bundles and deals on higher ticket items, in order to incentivize them to increase their spending. 

### Cluster M&F : 
These represent the highest value customers showing both high monetary value and frequency, though their recency lacks. The company should create VIP programs tailored to these individuals in order to retain them as high value customers.

