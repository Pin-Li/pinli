## Customer Value Segmentation using R

**Project Description:** We need to distinguish potential high-value consumers from low-value consumers according to their recency, frequency and monetary value (RFM). Recency refers to the number of months since the last month with purchase. Frequency is the sum of trips that a given individual made in the previous quarter. Monetary value is the average monthly expenditure for a consumer in the previous months. Then we apply different weights to these three factors and calculate an RFM index. At last we segment customers with 10 even-sized groups from low to high index values. The very last two groups with high index values are customers we want to target.

### 1. Recency

Since recency is based on last month's purchase history, I have to use a for-loop even if it runs quite slowly. 

```Rscript
df$recency <- NA

for( i in 1:1000 ) {
    for( m in 2:18 ) {
      if (df$count[df$id == i & df$monthnum == m-1] == 0) {
        df$recency[df$id == i & df$monthnum == m] <- df$recency[df$id == i & df$monthnum == m-1] + 1
      }else{df$recency[df$id == i & df$monthnum == m] <- 1}

    }
}
```

### 2. Frequency

```Rscript
index.quar <- data.frame("monthnum" = 1:18, quar = rep(c(1:6), each = 3) )
df <- merge(df, index.quar, by = "monthnum")
df

df$frequency <-NA
for( i in 1:1000) {
  for (m in 2:6) {
    
    df$frequency[df$id == i & df$quar == m] <- 
      sum(df$count[df$id == i & df$quar == m-1])
  }
}
```

### 3. Monetary Value

```Rscript
df$monvalue <- NA
for( i in 1:1000 ) {
  for ( m in 2:18 ) {    
    if( df$expd[ df$id == i & df$monthnum == m-1 ] == 0) { #if no purchase last month
      df$monvalue[df$id == i & df$monthnum == m] <- df$monvalue[ df$id == i & df$monthnum == m-1 ] #then monvalue unchanged
    }else{ 
      df$monvalue[df$id == i & df$monthnum == m] <- 
      sum(df$expd[df$id == i & df$monthnum < m])/ length(df$monthnum[df$id == i & df$monthnum < m & df$expd > 0])
    }
  }
}
```

### 4. RFM index

We are assuming that someone else (may be our boss) has told us the weights, so here we just simply add them up.
Using the equation: RFM =b1R+b2F+b3M

```Rscript
b1 <- -0.05
b2 <- 3.5
b3 <- 0.05
df$index <- b1*df$recency + b2*df$frequency + b3*df$monvalue
```

### 5. Validation

```Rscript
v<-rep(NA,10)
names(v)<-c('1','2','3','4','5','6','7','8','9','10')

df<-df[order(df$index),]
df.sort<-df[!is.na(df$index),]

cuts <- quantile(df$index, seq(0.1,1,0.1), na.rm = T)
cuts
v[1] <- mean (df.sort$expd [df.sort$index <= cuts[1]] ) 

for( i in 2:10 ){
  v[i] <- mean(df.sort$expd [df.sort$index <= cuts[i] & df.sort$index > cuts[i-1]])
}

barplot(v,xlab = 'deciles in the RFM index', ylab = 'average expenditure', ylim=c(0,20), main = 'Average expenditure by deciles in the RFM index')
```

### Conclusion
This is just an exercise for for-loop in R but I learnt a lot from this. Not only did I learn about the logic within for-loop, but also the importance of business sense. One would be lost in the real working scenario when he/she has to figure out "which customer is more valuable to my company". From this point, a machine would be less helpful and that is where we human could not be replaced.


