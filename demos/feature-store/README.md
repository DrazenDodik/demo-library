# Feature Store
A central repository storing curated, documented, and controlled access features useful for training and inferencing machine learning models.

### Why Feature Store? The main problem solved.

**Training-Serving Skew**
* During Training:
  * Models learn from processed data with transformations, scaling, and feature engineering. 
  * Predictions are based on the learned features.
* During Prediction:
  * New data predictions require the same preprocessing steps. 
  * Inconsistent preprocessing causes training-serving skew. 
  * Skew results in less accurate predictions compared to training.
  
<p align="center">
<img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/0*auWJxG-8AEve7Kzb.png"width="750", alt="Model development without Feature Store"/>
</p>

**Key Points on "Data Engineering is the Hardest Problem in Machine Learning":**

1. **Data's Crucial Role:** Handling data is the toughest and most vital part of machine learning (ML).
2. **Time-Intensive Tasks:** ML experts spend a lot of time choosing features, transforming them, and setting up pipelines for putting models into action.
3. **Common Data Hurdles:** Flawed data often causes issues when ML systems are used in the real world.
4. **Shift from Model Creation:** Using ML at a large scale is quite different from creating models initially.
5. **Focus on Feature Work:** Much effort is dedicated to tweaking features and organizing data during model development.
6. **Feature Techniques:** Methods like converting data types, normalization, and one-hot encoding are commonly used in this process.
7. **Deep Learning Impact:** With deep learning, using more data improves performance, but it adds complexity for data engineers.
8. **Training Trend:** ML is moving towards using bigger datasets, making scalability and efficiency crucial.
9. **Need for Standardization:** Standard and scalable platforms are necessary to handle the increasing complexity of working with data, especially as datasets get larger.

### Life Before Feature Store:

1. **Code Duplication:** Features are duplicated across training jobs, leading to non-DRY (Don't Repeat Yourself) code.
2. **Inconsistent Implementations:** Different implementations for computing features during training and deployment, causing potential prediction problems.
3. **Lack of Reusability:** Features embedded in training/serving jobs are not reusable, requiring Data Scientists to write low-level code for data access.
4. **Data Engineering Skills Required:** Data Scientists need data engineering skills to access data stores, hindering productivity.
5. **No Feature Management:** No centralized service for managing, searching, or governing features. Features are not treated as assets.

<p align="center">
<img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/0*JBzJL61M6zffg7wv.png" width="750", alt="Model development without Feature Store"/>
</p>

### Life with Feature Store:

1. **Unified Feature Repository:** Centralized platform for storing, retrieving, and managing features in an organized manner.
2. **Search and API Support:** Data Scientists can easily search for features and use them with API support, reducing data engineering complexity.
3. **Caching and Reusability:** Features can be cached and reused by multiple models, reducing training time and infrastructure costs.
4. **Managed and Governed Assets:** Features become managed, governed assets within the organization.
5. **Collaboration and Productivity:** Enhances collaboration by providing a standardized way to share features across teams and models.
6. **Economies of Scale:** Centralized feature storage facilitates an economies-of-scale effect, making it easier and cheaper to build new models.
7. **Reduction in Technical Debt:** Minimizes technical debt associated with ad-hoc feature engineering and complex training pipelines.

<p align="center">
<img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/0*1tko2Cxqw2jYyGza.png" width="750", alt="Model development using Feature Store"/>
</p>

### Feature Store Necessity: Key Questions

1. **What kind of features do your ML applications need?**
* _Use Feature Store:_ 
  * ML applications heavily relying on low-latency streaming features, where online feature pre-computation can significantly improve serving speed.
* _Don't Use Feature Store:_ 
  * ML applications that predominantly use batch features, and there is no stringent requirement for low-latency serving.

2. **What type of ML applications do your organizations manage?**
* _Use Feature Store:_
  * ML applications with operational use cases (e.g., fraud detection, recommendation) that require streaming features and low-latency serving.
  * Batch training + online inference scenarios with low serving latency and a significant number of features to be computed on the fly.
  * Organizations with multiple data science teams that need to share and reuse features to enhance collaboration.
* _Don't Use Feature Store:_ 
  * ML applications with batch feature engineering + batch inference, where there's no need for streaming features or low-latency serving.

3. **Is there a need to share and reuse features among various teams in your organization?**
* _Use Feature Store:_ 
  * Organizations with multiple data science teams where sharing and reusing features can significantly improve collaboration and reduce duplicated effort.
* _Don't Use Feature Store:_ 
  * Organizations where feature sharing among teams is not a priority or where there is a minimal need for collaboration.

5. **Is training-serving skew often an issue that negatively impacts ML model performance?**

* _Use Feature Store:_ 
  * If training-serving skew is a common issue impacting ML model performance, a feature store can help ensure consistency between training and serving phases.
* _Don't Use Feature Store:_ 
  * If training-serving skew is not a significant concern and the ML model performs consistently in both training and serving environments.



### Sources / Useful links
1. [Feature Stores: the missing Data Layer for ML Pipelines](https://jim-dowling.medium.com/feature-stores-the-missing-data-layer-for-ml-pipelines-728e0102aad8)
2. [Do you really need a Feature Store?](https://towardsdatascience.com/do-you-really-need-a-feature-store-f71cf9586158)
