## Datascience tasks
- Data extraction
- Data analysis (EDA)
- Data preperation (cleaning, train-test split)
- Model training (training, hyperparam tuning)
- Model evaluation (metrics to judge it)
- Model validation (is its predictive capability higher than a baseline?)
- Model serving (deployement REST API, embedded model etc)
- Model monitoring (predictive capabilities checked to know if action needed)

## MLOps lvl 0

The level of automation in these tasks shows how mature the mlops process is. At lvl 0, once deployed, the model isn't monitored and there's no CI since its assumed that it's final. Models often degrade during prodiction though (eg due to concept drift (the inputs change with time) or schema changes (the input format itself is different))

## MLOps lvl 1

![[Pasted image 20230702004906.png]]

In lvl 0 you deploy the model, here you deploy a pipeline that trains the model automatically. This requires the use of triggers with
- Data validation (schema and values)
- Model validation (performance and compatibility)

### Feature Stores

These can be implemented as a repository of features. So you don't have to waste time rerunning the data transormation steps everytime you open a notebook. These are shared across teams so if someone else already made the features you need you save time. And since all features have a common documentation there's no implementation inconsistensies.

