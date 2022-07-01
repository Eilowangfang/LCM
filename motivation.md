# Choose the right estimators for your query!

## Existing estimators can be divided into two categories: traditional and learning-based.

Traditional (histogram, sampling) methods often have fast estimation but lower accuracy.

Data-driven learning methods often have high accracy but lone estimation.

Query-driven learning methods often have low accracy but fast estimation. 


## Motivation 1: Both inference time and estimation accuracy count
![image](https://user-images.githubusercontent.com/52020936/176805613-384f8c60-9901-4c60-87cb-6c4b281f4d42.png)


## Motivation 2: Accuracy of estimators are not consistent across different queries
![image](https://user-images.githubusercontent.com/52020936/176805990-26490ac6-cca1-49d5-9316-e499825eac6c.png)



## Motivation 3: Bad estimations may also lead to good execution plan
![image](https://user-images.githubusercontent.com/52020936/176805904-4a6961e9-095a-447b-9363-9caa1c63ac16.png)



## Propose: Consider multiple estimators for your query optimization!
![image](https://user-images.githubusercontent.com/52020936/176806060-eab940b5-45e3-4718-b27a-e4bd98907c68.png)

