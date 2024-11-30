# CorrelationExplorer
Compare the Correlation of Metabolites across different LC-MS methods
Metabolomics data in the last 10 years has become a powerful tool in understanding disease. It adds insight into the complicated intermediates and dysregulated pathways that can act as a thumb print of disease. My lab takes a multi-method untargeted liquid chromatography mass spectrometry (LC-MS) approach to deciphering this unique print. Each analytical method provides a piece of the metabolome optimized by compound polarity, common functional groups and structural similarity (Clish, 2015). This patchwork approach allows for better coverage and more optimized untargeted methods. Unsurprisingly, within this patchwork there is overlap. This can provide an opportunity for certainty in metabolite annotation as well as confirmation of trends across samples in a study. Our annotation library has both putatively annotated compounds as well as standard confirmed. While running a study, our current approach for annotation is through retention time (RT) and mass-to-charge (MZ) matching through manual inspection against our annotation library. We run large cohort studies across all our methods and have an annotation library upwards of 3000 annotations. When metabolite trends correlate across different chromatography methods, it aids in our certainty of annotation as well as shows the robustness of our analytical methods. Previously, this quality control (QC) step was done by hand on final datasets. When looking through thousands of metabolite annotations and sometimes thousands of samples, it can be tedious, time consuming and prone to error. To address this problem, I have created an R Shiny app that looks at correlations across similar metabolites annotated in different methods for a given dataset. The expectation is this app will make this important QC step easier and provide important insight and certainty in our analytic methodology. 


Methods: 
Once the data is acquired, we have created an in-house pipeline for aligning, annotating, and drift correcting data (Hitchcock, 2023). Our final output is a extensive excel sheet. Each project will have one of these robust excel sheets for each method run. Because of this pipeline, the format of each sheet is similar which is of course helpful. 'Metabolite' is the last column before sample data starts. Each sheet contains columns with the SuperClass, MainClass and SubClass for each metabolite annotated, Mass-to-Charge ratio (MZ), Retention Time (RT) and coefficient of variance (CV) calculation for our quality control (QC) samples. There are many other columns that are also included that are necessary but not required for this app. The column content will of course vary depending on the analytical method and what annotations were found in the specific samples. After the Metabolite column, is the sample data. While every sample is expected to be run for each method, the order they run in may vary. This can be due to samples needing to be re-injected or re-extracted during the run. Our pipeline lists the samples by our run-order. This is key for drift correction as well as PCA plots that ensure there are no trends in the data based on extraction and injection order. Our pipeline also inserts any pertinent metadata information in the rows above. To make this app as user friendly as possible, the only change that needs to happen to these sheets is that the rows with metadata need to be removed so that the first row contains the sample names and other column information. It will also need to be saved as a CSV and ensure metabolite is the first column before the sample information. Once the sheets are in correct format (CSV, metabolite as column right before data and there is no metadata above), the Correlation Explorer can be run. 



The first step of this app is to format and filter the CSV files. The files are read in and saved as 'File1' and 'File2'. The files maintain this nomenclature throughout the code. It then defines the 'start column' as the ‘Metabolite’ column so that we know where the data from the samples begin. It first checks that each sheet has unique metabolite names. This is an important step for us as we use two different types of software for peak extraction. This may result in duplicate names appearing. While this is unlikely with our pipeline, it will throw off the correlations if there are multiple metabolites with the same name. It then confirms that there are in fact metabolites that overlap between the sheets. It then filters each data frame to only contain the overlapping metabolites. 
The app then goes through a similar process with sample names. It checks that there are not duplicated sample names in each file and then, that there exist overlapping samples between the files. It then filters the files to only contain the overlapping file names. There is also a step where it removes our QC samples. This QC sample is a pool that contains a small amount from every sample run for a given study. We run two of these pooled samples every 20 study samples. This acts as a QC for our sample runs and assess instrument function. We utilize these pools during our drift correction as well. It is important to pull these out of our correlations so as not to skew the value.
 A data frame is created for each of the files to contain only the overlapping metabolites and samples. Each file is then transposed, and a correlation is run. The correlation uses only complete pairwise observation which is important to account of any missingness for a sample for a given metabolite. Because of the nature of the data, I chose a Pearson correlation. The data is continuous and assumed to be normally distributed. Our QC allows for detection of outliers and in this case, we are looking at a linear relationship between the metabolites. 


Results: 
The information of this correlation is displayed in several different ways. This allows the user to have different options in assessing the correlations and understanding their significance. The first tab is a “Correlation Table”. This table gives you a breakdown of the number of overlapping metabolites within specific thresholds. For example, the number of metabolites that have a correlation less than 0.85. It also gives you a percentage, allowing you to quickly see what percent of the metabolites that overlap have a correlation below 0.85. This quick visualization can help you determine overall how well these datasets correlated across matching metabolites. 
 
The next tab is a “Correlation Table by Metabolite” and contains more detailed information showing each metabolite basis across both files. This table is ordered by the correlation value under the assumption it is these lower correlation metabolites you will want to further inspect. There is a column with the metabolite name and correlation value. There are then subsequent columns pulled from each file. This includes the CV of the pooled sample. As mentioned above, we use two pooled samples run every 20 study samples. While they provide different uses throughout our processing pipeline, at this stage of processing one is used in drift correction, and one acts as a QC for the metabolite. I have included in the table the CV value of the QC pool for each file for a given metabolite. I also included the Row Median for that metabolite, the RT and the MZ denoted with which file they came from. Evaluating these different parameters can help give insight into a correlation value. For example, if the row median is super low, it might indicate that this feature could not resolve well from the baseline and is at the limit of detection for that method. This would explain the low correlation and we would want to only report it in the method that can better resolve this feature. Additionally, if the RT in one method is much earlier in the chromatogram (i.e. in the first minute), we know that this part of the chromatogram can be very busy with elusion off the column- which can further indicate why the metabolite is not well correlated in a method that can resolve it later in the gradient. This table can help provide insight into correlation values and helps the individual make an important call of removing an annotation from one of the methods if there is an indication of poor identification. 

 

The next visualization is a plot graphing the metabolite from each file against one another. If there was a perfect correlation you would expect this to give you a straight line. These plots can also help you understand what is causing low correlation. These plots are organized by class of metabolite. Both the Pearson and Spearman correlations are given at the top of the plot here. Showing both can give better validation in the correlation and visualizing the data this way may indicate Spearman would be a better correlation choice (i.e. if there were a lot of outliers in a given metabolite). In testing these plots with different datasets, it turned out to also be a powerful tool for finding the limit of detection. In cases where a drug metabolite was being measured for example you could see a cloud of samples until you hit a certain relative abundance and then the metabolites map perfectly together. This graphical visualization of the correlation is important in helping to diagnose which samples are driving the correlation. You can select which metabolite you would like to look at from the drop-down menu on the left-hand panel. The example below shows DG 36:2 which has a good Pearson correlation of 0.8844, this is shown graphical in the data. Palmitoyl-EA however has a low Pearson correlation of 0.1039, and the samples show a cloud clustering. 

 

 

 
The next visualization is a “Correlation Heatmap”. This heatmaps shows the correlation matrix from running a Pearson correlation across the files.  On the outside of the heatmap you can see the color bars indicating the specific class of metabolites. This heatmap is helpful only in further validating what we found in the previous information. The advantage of this visualization is that while the diagonal gives you the correlation between the specific metabolites, the color bars allow you to look at the correlation within a class of lipids. For example, you would expect those of a similar class to be more correlated with each other than those of a different class. You can notice that where the class overlaps, there is a higher correlation shown in the darker red boxes. This can further help you feel confident in the annotations chosen. If this was not the case, you might question the validity of the annotation or how well that class of metabolites resolves in a specific method. 







 There are then correlation heatmaps for each individual file. This again just provides a visualization of annotation and confirmation that each feature is more correlated with those of its similar class then another. While seemingly simple, this can be very important for our annotation practices. 
 
 


Conclusion
The purpose of this app is to provide an easily implemented QC step to the processing pipeline of Metabolomics data. The value it brings to our current pipeline is the comparison across chromatography methods to redundant metabolite annotations. This provides us with validation in feature identification, confidence in data trends, and insurance that the highest quality feature annotated is reported. The next step after running this application and finding which metabolites are not correlated, would be to manually pull raw files and visually inspect the extracted chromogram to understand the quality of the peak. While the row median, CV of pool and RT can give you some indication, manual inspection is required to be thorough. Building in a feature that can read in raw files and extract the feature would be the next step for this type of tool. My expectation with this application is to provide my team with an easily implemented app that can aid in our rigorous QC pipeline. 

 




Citations 
Clish CB. Metabolomics: an emerging but powerful tool for precision medicine. Cold Spring Harb Mol Case Stud. 2015 Oct;1(1):a000588. doi: 10.1101/mcs.a000588. PMID: 27148576; PMCID: PMC4850886.
Daniel S. Hitchcock, Jesse N. Krejci, Courtney A. Dennis, Sarah T. Jeanfavre, Julian R. Avila-Pacheco,  Clary B. Clish. Eclipse: Alignment of Two or More Nontargeted LC-MS Metabolomics Datasets using Direct Subalignments. bioRxiv. 2023 June 11., doi: https://doi.org/10.1101/2023.06.09.544417



















