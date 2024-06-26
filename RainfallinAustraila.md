---
title: "DS 805 Final Project"
author: "Sam, Kindyl, Kajal"
---
```{r}
library(quantmod)
library(imputeTS)
library(ipred)
library(rpart.plot)
library(Metrics)
library(caret)
library(rpart)
library(randomForest)
library(tidyverse)
library(mice)
library(cowplot)
library(skimr)
library(tidyverse)
library(ggplot2)
library(pROC)
library(class)
```


```{r}
set.seed(80)
df_full <- read.csv("C:/Users/svolp/OneDrive/Desktop/Upper Level Classes/DS 805 Statistical Learning/weatherAUS.csv", sep = ",", header = TRUE)

df=na.omit(df_full)

#Cant have NA's for the response variable
df <- df[!is.na(df$RainTomorrow), ]

# Manipulating the data
df$RainTomorrow <- ifelse(df$RainTomorrow == "No", 0, 1)
df$RainToday <- ifelse(df$RainToday == "No", 0, 1)
df$Date <- as.POSIXct(df$Date)
```

## Part 1: Exploratory Data Analysis

1.  Check for existence of NA's (missing data), if necessary, impute the missing data.

    ```{r}
    # Checking the number of NA's as well as the proportion missing
    columns_w_na <- df %>%
      summarise(across(everything(), ~sum(is.na(.)))) %>%
      pivot_longer(cols = everything(), names_to = "column", values_to = "missing_values") %>%
      filter(missing_values > 0) %>%
      mutate(proportion_of_na = round(missing_values / sum(missing_values), 2))
    columns_w_na

    dim(df)

    table(df$RainTomorrow)
    ```

2.  Classify all categorical variables as factors. Calculate the summary statistics of the entire data set.

    ```{r}
    df <- df %>% 
      mutate(RainToday = ifelse(RainToday == "No", 0, 1),
             Location = as.factor(Location),
             WindGustDir = as.factor(WindGustDir),
             WindDir9am = as.factor(WindDir9am),
             WindDir3pm = as.factor(WindDir3pm))

    glimpse(df)
    summary(df)

    # levels(df$Location)
    # levels(df$WindGustDir)
    # levels(df$WindDir9am)
    # levels(df$WindDir3pm)

    ```

3.  For the numerical variables, plot box plots based on values of `y`.

    ```{r}
    # Variables of interest
    # 'MinTemp', 'MaxTemp', 'Rainfall', 'Evaporation', 'Sunshine','WindSpeed9am', 'Humidity3pm', 'Pressure3pm', 'Cloud9am', 'Cloud3pm', 'Temp3pm'

    MinTemp_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = MinTemp)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of MinTemp by RainTomorrow"), x = "Rain Tomorrow", y = "MinTemp") +
        theme_bw()

    MaxTemp_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = MaxTemp)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of MaxTemp by RainTomorrow"), x = "Rain Tomorrow", y = "MaxTemp") +
        theme_bw() 

    Rainfall_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Rainfall)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Rainfall by RainTomorrow"), x = "Rain Tomorrow", y = "Rainfall") +
        theme_bw() 

    Evaporation_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Evaporation)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Evaporation by RainTomorrow"), x = "Rain Tomorrow", y = "Evaporation") +
        theme_bw() 

    Sunshine_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Sunshine)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Sunshine by RainTomorrow"), x = "Rain Tomorrow", y = "Sunshine") +
        theme_bw() 

    WindSpeed9am_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = WindSpeed9am)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of WindSpeed9am by RainTomorrow"), x = "Rain Tomorrow", y = "WindSpeed9am") +
        theme_bw() 

    Humidity3pm_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Humidity3pm)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Humidity3pm by RainTomorrow"), x = "Rain Tomorrow", y = "Humidity3pm") +
        theme_bw() 

    Pressure3pm_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Pressure3pm)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Pressure3pm by RainTomorrow"), x = "Rain Tomorrow", y = "Pressure3pm") +
        theme_bw() 

    Cloud9am_Boxplot <-  ggplot(df, aes(x = factor(RainTomorrow), y = Cloud9am)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Cloud9am by RainTomorrow"), x = "Rain Tomorrow", y = "Cloud9am") +
        theme_bw() 

    Cloud3pm_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Cloud3pm)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Cloud3pm by RainTomorrow"), x = "Rain Tomorrow", y = "Cloud3pm") +
        theme_bw() 

    Temp3pm_Boxplot <- ggplot(df, aes(x = factor(RainTomorrow), y = Temp3pm)) +
        geom_boxplot() +
        labs(title = paste("Boxplot of Temp3pm by RainTomorrow"), x = "Rain Tomorrow", y = "Temp3pm") +
        theme_bw() 

    # Combine the list of plots into one object
    combined_plot <- plot_grid(MinTemp_Boxplot, MaxTemp_Boxplot, Rainfall_Boxplot, Evaporation_Boxplot, Sunshine_Boxplot, WindSpeed9am_Boxplot, Humidity3pm_Boxplot, Pressure3pm_Boxplot, Cloud9am_Boxplot, Cloud3pm_Boxplot, Temp3pm_Boxplot)

    # Display the combined plot
    combined_plot
    ```

4.  For the categorical variables, plot bar charts for the different values of `y`.
    ```{r}
    # Location WindGustDir WindDir9am WindDir3pm

    # Variables of interest
    categorical_variables <- c('Location', 'WindGustDir', 'WindDir9am', 'WindDir3pm')

    # Function to create bar charts
    create_bar_chart <- function(variable) {
      # Create a data frame for the specific variable
      df_variable <- df %>% 
        group_by(RainTomorrow, !!sym(variable)) %>% 
        summarise(count = n()) %>%
        filter(!is.na(!!sym(variable))) %>%
        mutate(RainTomorrow = factor(RainTomorrow, levels = c(0, 1)))
      
      # Plot the bar chart
      ggplot(df_variable, aes(x = !!sym(variable), y = count, fill = RainTomorrow)) +
        geom_bar(stat = "identity", position = "dodge") +
        labs(title = paste("Bar Chart of", variable, "by RainTomorrow"), x = variable, y = "Count") +
        theme_bw() +
        theme(axis.text.x = element_text(angle = 45, hjust = 1, size = 4))
    }

    # Create and store bar charts for each categorical variable
    bar_charts <- lapply(categorical_variables, create_bar_chart)

    # Combine the list of bar charts into one object
    combined_bar_chart <- plot_grid(plotlist = bar_charts, align = "v")

    # Display the combined bar chart
    print(combined_bar_chart)

    ```

5.  Test/training separation

    ```{r}
    set.seed(80)
    n1=nrow(df)
    nt1=round(nrow(df)*.8)
    indexes = sample(n1, nt1)
    df_train = df[indexes,]
    df_test = df[-indexes,]

    # train_locations <- table(df_train$Location)
    # test_locations <- table(df_test$Location)
    ```

## Part 2: Logistic Regression or LDA

1.  Develop a classification model where the variable `y` is the dependent variable using the Logistic Regression or LDA, rest of the variables, and your training data set.

    ```{r}
    # Logistic Regression with all variables
    log.model<-glm(RainTomorrow~., data=df_train, family=binomial)
    summary(log.model)

    # Logistic Regression Predictions
    logprob<-predict(log.model, newdata=df_test, type="response")
    head(logprob,3)
    ```

2.  Obtain the confusion matrix and compute the **testing error rate** based on the logistic regression classification.

```{r}
#contrasts(factor(df_test[,"RainTomorrow"]))
logpred=rep(0, nrow(df_test))
logpred[logprob>=.5]=1
logpred=as.factor(logpred)
head(logpred,3)

cm=confusionMatrix(data=logpred, reference=as.factor(df_test$RainTomorrow))
cm
    
```

```{{r}}
# Create ROC curve
roc_curve <- roc(df_test$RainTomorrow, logprob)

# Plot ROC curve
plot(roc_curve, main = "Logistic ROC Curve", col = "blue")

# Add labels and legend
legend("bottomright", legend = paste("AUC =", round(auc(roc_curve), 2)), col = "blue", lty = 1, cex = 0.8)

```


## Part 3: KNN

1.  Apply a KNN classification to the training data using.

    ```{r}
    df_traink <- df_train %>% 
      mutate(
        Location = as.numeric(as.factor(Location)),
        WindGustDir = as.numeric(as.factor(WindGustDir)),
        WindDir9am = as.numeric(as.factor(WindDir9am)),
        WindDir3pm = as.numeric(as.factor(WindDir3pm)))

    df_testk <- df_test %>% 
      mutate(
        Location = as.numeric(as.factor(Location)),
        WindGustDir = as.numeric(as.factor(WindGustDir)),
        WindDir9am = as.numeric(as.factor(WindDir9am)),
        WindDir3pm = as.numeric(as.factor(WindDir3pm)))

    df_traink <- na.omit(df_traink)
    df_testk <- na.omit(df_testk)

    knn.train <- df_traink[, 2:18]
    knn.test <- df_testk[, 2:18]

    knn.trainLabels <- df_traink[,"RainTomorrow"]
    knn.testLabels <- df_testk[,"RainTomorrow"]

    set.seed(80)
    k.grid=1:10
    error=rep(0, length(k.grid))

    for (i in seq_along(k.grid)) {
      pred = knn(train = scale(knn.train), 
                 test  = scale(knn.test), 
                 cl    = knn.trainLabels, 
                 k     = k.grid[i])
      error[i] = mean(knn.testLabels !=pred)
    }

    min(error)

    knn9 <- knn(train = knn.train, test = knn.test, cl = knn.trainLabels, k = 9)

    summary(knn9)
    ```

2.  Obtain the confusion matrix and compute the testing error rate based on the KNN classification.

    ```{r}
    confusionMatrix(knn9, as.factor(knn.testLabels))
    ```

    ```{r}
    # Compute distances between test data and training data
    distances <- as.matrix(dist(scale(knn.test, center = colMeans(knn.train), scale = apply(knn.train, 2, sd))))

    # Find the k-nearest neighbors
    nearest_neighbors <- apply(distances, 1, order)[, 1:9]

    # Get the labels of the k-nearest neighbors
    nearest_labels <- knn.trainLabels[nearest_neighbors]

    # Count the number of "Yes" and "No" labels among the nearest neighbors
    counts <- t(sapply(1:nrow(nearest_labels), function(i) table(nearest_labels[i, ])))

    # Compute probabilities
    prob_yes <- counts[, "Yes"] / 9  # Assuming k = 9
    prob_no <- counts[, "No"] / 9     # Assuming k = 9

    # Combine probabilities for both classes
    predicted_probs <- cbind(Yes = prob_yes, No = prob_no)

    # Create ROC curve
    library(pROC)
    knn_roc_curve <- roc(as.numeric(knn.testLabels) - 1, predicted_probs[, "Yes"])

    # Plot ROC curve
    plot(knn_roc_curve, main = "KNN ROC Curve", col = "red")

    # Calculate AUC
    knn_auc <- auc(knn_roc_curve)

    # Add AUC value to the plot
    legend("bottomright", legend = paste("AUC =", round(knn_auc, 2)), col = "red", lty = 1, cex = 0.8)

    ```

```{r}

predicted_probs <- attr(knn9, "prob")

# Create ROC curve
knn_roc_curve <- roc(df_test$RainTomorrow, predicted_probs[, 2])

# Plot ROC curve
plot(knn_roc_curve, main = "KNN ROC Curve", col = "red")

# Calculate AUC
knn_auc <- auc(knn_roc_curve)
legend("bottomright", legend = paste("AUC =", round(knn_auc, 2)), col = "red", lty = 1, cex = 0.8)
```

3.  Explain your choices and communicate your results.

## Part 4: Tree Based Model

1.  Apply one of the following models to your training data: *Random Forest, Bagging or Boosting*

```{r}
#Bagged model Project data
model.bag=bagging(factor(RainTomorrow) ~ ., data=df_train, coob = TRUE)
model.bag
```

2.  Obtain the confusion matrix and compute the testing error rate based on your chosen tree based model.

```{r}
# predictions
pred.bag = predict(model.bag, newdata =df_test, type="class")

# confusion matrix
cm.bag <- confusionMatrix(pred.bag, as.factor(df_test$RainTomorrow))
cm.bag
```

3.  Explain your choices and communicate your results.

```{r}

bag.pred_probs = apply(predict(model.bag,newdata=df_test, type = "prob"), 1, max)

# Assuming pred.bag contains the binary predictions (0s and 1s)
# Calculate ROC curve
bag_roc_curve <- roc(df_test$RainTomorrow, bag.pred_probs)

# Plot the ROC curve
plot(bag_roc_curve, main = "ROC Curve", col = "blue", lwd = 2)

# # Add diagonal reference line
# abline(a = 0, b = 1, lty = 2)

# Add AUC to the plot
text(0.8, 0.2, paste("AUC =", round(auc(bag_roc_curve), 2)))

```

```{r}
# Make a Random Forest Model (Imbalanced data)
#RF
model.rf=randomForest(factor(RainTomorrow) ~ . , data = df_train , importance=TRUE,proximity=TRUE)

pred.rf = predict(model.rf, newdata =df_test, type="class")

# confusion matrix
cm.rf <- confusionMatrix(pred.rf, as.factor(df_test$RainTomorrow))
cm.rf

rf.pred_probs=apply(predict(rf.mod, newdata = df_test, type = "prob"), 1, max)



```

```{r}
rf.pred_probs=apply(predict(rf.mod, newdata = df_test, type = "prob"), 1, max)

# Random Forest Roc Curve
rf_roc_curve <- roc(df_test$RainTomorrow, rf.pred_probs)

# Plot the ROC curve
plot(rf_roc_curve, main = "ROC Curve", col = "blue", lwd = 2)

# Add diagonal reference line
abline(a = 0, b = 1, lty = 2)

# Add AUC to the plot
text(0.8, 0.2, paste("AUC =", round(auc(rf_roc_curve), 2)))

```

## Part 5: SVM 

1.  Apply a SVM model to your training data.

```{r}
library(e1071)
svm_model<- svm(factor(RainTomorrow) ~ ., data = df_train, type = "C-classification", kernel = "radial", scale = FALSE)
svm_model
```

2.  Calculate the confusion matrix using the testing data.

```{r}
pred_test <- predict(svm_model, df_test)
confusionMatrix(factor(pred_test, levels=0:1), factor(df_test$RainTomorrow))
```

3.  Explain your choices and communicate your results.

```{r}
set.seed(80)
separator =  sample(1:2,size=nrow(Wine4), prob=c(0.80, 0.20), replace=TRUE) 
train4=Wine4[separator == 1, ]
test4=Wine4[separator == 2, ]
test4$class=test4$binary
test4=subset(test4, select=-binary)

df_test
```

```{r}
set.seed(1234)
#library(smotefamily)
new_train<- SMOTE(X=df_train[,-23], target=df_train[,23], K=5)
table(new_train$data$class)
```

```{r}
set.seed(1234)
train4.1=new_train$data
train4.1$class=as.numeric(train4.1$class)
data1=as.matrix(train4.1)
data1.t=as.matrix(test4)

ddata1 <- xgb.DMatrix(data1, label=as.numeric(train4.1$class))
ddata1.t <- xgb.DMatrix(data1.t, label=test4$class)

Boos1 <- xgb.train(data = ddata1, max_depth = 2,
              eta = 1, nthread = 4, nrounds = 5,
              watchlist = list(train = ddata1, eval = ddata1),
              objective = "binary:logistic")
```

```{r}
pred.1=ifelse(predict(object = Boos1,ddata1.t)>0.5,"1","0")
c1=confusionMatrix(reference=factor(test4$class),
                          data=factor(pred.1))
c1
```

```{r}
#### ROC CURVES

# TREE
tree.mod = rpart(binary~., train, method="class", parms=list(split="gini"))
tree.pred=apply(predict(tree.mod, newdata=test), 1, max)

# Pruned Tree
cp_opt=tree.mod$cptable[which.min(tree.mod$cptable[, "xerror"]), "CP"]
pruned.mod= prune(tree = tree.mod, cp = cp_opt)
pruned.pred=apply(predict(pruned.mod, newdata=test), 1, max)

#Bagging
set.seed(123)

bag.mod=bagging(factor(binary) ~ ., data=train, coob = TRUE,
                   nbagg=(nrow(train)/2))
bag.pred = apply(predict(bag.mod,newdata=test, type = "prob"), 1, max)

#RF
rf.mod=randomForest(factor(binary) ~ . , data = train , importance=TRUE,proximity=TRUE)
rf.pred=apply(predict(rf.mod, newdata = test, type = "prob"), 1, max)

# List of predictions
pred.list=list(tree.pred, pruned.pred, bag.pred, rf.pred, pred.1)

# List of actual values
nmod=length(pred.list)
actual=rep(list(test$binary), nmod)

# Plot the ROC curves
#library(ROCR)
pred.r=prediction(pred.list, actual)
roc.r=performance(pred.r, "tpr", "fpr")
plot(roc.r, col = as.list(1:nmod), main = "ROC Curves: Test Set")
legend(x = "bottomright", 
       legend = c("Decision Tree", "Pruned Tree", "Bagged Trees", "Random Forest", "Boosting"),
       fill = 1:nmod)
```
