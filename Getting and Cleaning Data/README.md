### README for run_analysis.R


The script run_analysis.R executes the following steps and ends with the creation of a tidy datafile called "tidy.txt".

### 0. Download and unzip the UCI HAR Dataset (Human Activity Recognition Using Smartphones Dataset Version 1.0), then move to the working directory
```{r}
library(dplyr)
filename <- "UCI HAR Dataset.zip"

if (!file.exists(filename)){
        fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
        download.file(fileURL, filename, method="curl")
}  
if (!file.exists("UCI HAR Dataset")) { 
        unzip(filename)
}
setwd('./UCI HAR Dataset/')
```

### 1. Merge the accelerometer training and test sets to create one data set
```{r}
labels_train <- read.table("./train/y_train.txt", col.names = "label")
subject_train <- read.table("./train/subject_train.txt", col.names = "subject")
dataset_train <- read.table("./train/X_train.txt")
train <- cbind(labels_train,subject_train,dataset_train)

labels_test <- read.table("./test/y_test.txt", col.names = "label")
subject_test <- read.table("./test/subject_test.txt", col.names = "subject")
dataset_test <- read.table("./test/X_test.txt")
test <- cbind(labels_test,dataset_test,subject_test)

data <- rbind(train,test)
```

### 2. Extract only the measurements on the mean and standard deviation for each measurement
```{r}
features <- read.table("features.txt", col.names=c("label","feature"))
extract <- (grep("(mean|std)", features[,2])) + 2
data_extract <- data[, c(1,2,extract)]
```

### 3. Use descriptive activity names to name the activities in the data set
```{r}
activities <- read.table("activity_labels.txt")
data_extract <- mutate(data_extract, label = activities[data_extract[, 1], 2])
```

### 4. Labels the data set with descriptive variable names
```{r}
featurecol <- as.character(features[extract,2])
names(data_extract) <- c("activity", "subject", featurecol)
```

### 5. Create tidy data set with the average of each variable for each activity and each subject
```{r}
result <- aggregate(data_extract[,3:81], by=list(activity=data_extract$activity, subject=data_extract$subject), FUN=mean)
write.table(result, "../tidy.txt")
```

