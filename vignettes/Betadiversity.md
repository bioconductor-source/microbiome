<!--
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteIndexEntry{microbiome tutorial - variability}
  %\usepackage[utf8]{inputenc}
  %\VignetteEncoding{UTF-8}  
-->
Beta diversity (microbiome heterogeneity)
-----------------------------------------

Load example data

    library(microbiome)
    data(peerj32)
    pseq <- peerj32$phyloseq

### Quantifying group heterogeneity / spread

Beta diversity within a given sample set can be quantified as the
average dissimilarity of each sample from the group mean. This was
applied in group-level comparisons for instance in [Salonen et al. ISME
J
2014](http://www.nature.com/ismej/journal/v8/n11/full/ismej201463a.html)
(they focused on homogeneity using inverse correlation, whereas here we
focus on heterogeneity using correlation but the measure is essentially
the same).

Calculate group diversities within the LGG (probiotic) and Placebo
groups

    b.pla <- group_diversity(subset_samples(pseq, group == "Placebo"))
    b.lgg <- group_diversity(subset_samples(pseq, group == "LGG"))

Use these to compare microbiota heterogeneity within each group. The LGG
group tends to have smaller values, indicating that the samples are more
similar to the group mean, and the LGG group is less heterogeneous (has
smaller spread / is more homogeneous):

    boxplot(list(LGG = b.lgg, Placebo = b.pla))

![](Betadiversity_files/figure-markdown_strict/heterogeneity-example2bbb-1.png)

The **inter- and intra-invididual stability** (or homogeneity) measures
are obtained as 1-b where b is the group diversity with the
anticorrelation method ([Salonen et al. ISME J
2014](http://www.nature.com/ismej/journal/v8/n11/full/ismej201463a.html)).

### Intra-individual heterogeneity

Quantify beta diversity within subjects over time. [Salonen et al. ISME
J
2014](http://www.nature.com/ismej/journal/v8/n11/full/ismej201463a.html))
used this to quantify intra-individual stability (ie homogeneity) of
subjects between two groups.

    betas <- list()
    groups <- as.character(unique(meta(pseq)$group))
    for (g in groups) {
      df <- meta(subset_samples(pseq, group == g))
      beta <- c()

      for (subj in df$subject) {
        # Pick the samples for this subject
        dfs <- subset(df, subject == subj)
        # Check that the subject has two time points
        if (nrow(dfs) == 2) {
          s <- as.character(dfs$sample)
          # Here with just two samples we can calculate the
          # beta diversity directly
          beta[[subj]] <- 1-cor(abundances(pseq)[, s[[1]]],
                            abundances(pseq)[, s[[2]]],
                    method = "spearman")
        }
      }
      betas[[g]] <- beta
    }

    boxplot(betas)

<img src="Betadiversity_files/figure-markdown_strict/homogeneity-example2c-1.png" width="300px" />

### Beta diversity within individual over time

Calculate change in beta diversity (community dissimilarity) over time
within a single individual

    data(atlas1006)
    pseq <- atlas1006

    # Identify subject with the longest time series (most time points)
    s <- names(which.max(sapply(split(meta(pseq)$time, meta(pseq)$subject), function (x) {length(unique(x))})))

    # Pick the metadata for this subject and sort the
    # samples by time
    df <- meta(subset_samples(pseq, subject == s)) %>% arrange(time)

    # Calculate the beta diversity between each time point and
    # the baseline (first) time point
    beta <- c(0, 0) # Baseline similarity
    s0 <- subset(df, time == 0)$sample
    for (tp in df$time[-1]) {
      # Pick the samples for this subject
      # If the same time point has more than one sample,
      # pick one at random
      st <- sample(subset(df, time == tp)$sample, 1)
      a <- abundances(pseq)
      b <- 1 - cor(a[, s0], a[, st], method = "spearman")
      beta <- rbind(beta, c(tp, b))
    }
    colnames(beta) <- c("time", "beta")
    beta <- as.data.frame(beta)

    p <- ggplot(beta, aes(x = time, y = beta)) +
           geom_point() + geom_line()
    print(p)       

<img src="Betadiversity_files/figure-markdown_strict/homogeneity-example2d-1.png" width="300px" />