<img align="center" src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/NC_Litigation_Banner.JPG" width="auto" height="auto">

The NC_Litigation_Predictor is a probabilistic classifier; it provides a probability of success of a motion for summary judgment when considering the features of a given lawsuit. The model applies machine learning, built upon all of North Carolina's appellate decisions, and provides a prediction grounded in data, based upon the user's judge, jurisdiction, and case type, to supplement the legal-threshold criteria, so that stakeholders can make an informed cost-benefit analysis on litigation strategy. The final model may be run <a href="https://share.streamlit.io/jnels13/nc_litigation_predictor/main/litigation_predictor_streamlit.py" target="_blank">HERE.</a>

This README follows the following format: 
<ul>
    <li> Repo Contents
    <li> Background
    <li> Process + Modeling
    <li> Results
</ul>

### Repo Contents
<ul>
    <li> README.md: Project description, processes, and outcome
    <li> Notebooks:
    <ul>
        <li> NC_COA_Scraper.ipynb: Jupyter notebook which indexes and downloads the initial set of appellate opinions (NC COA, 1998-2020)
        <li> DF_Creation.ipynb: Jupyter notebook which the opinions are converted from PDFs to a functional dataframe with minimal preprocessing
        <li> Label and Feature Creation.ipynb: notebook where labels and features are extracted from the appellate opinions
        <li> Litigation_Predictor.ipynb: Jupyter notebook containing the project model and results
    </ul>
    <li> Data:
    <ul>
        <li> SampleData: Folder containing sample data used throughout the project
        <li> ProjectData: Folder containing the project data (PDFs) and initial dataframes, NOT tracked in Git  
    </ul>
</ul>

### Background

#### The Common Law and Appellate Court's Opinions

Laws are composed of statutes/regulations (prepared by legislative and executive branches) and the common law. The common law is composed of written opinions (or rulings) of courts, typically courts of appeals, which review trial courts' application of the law to a given lawsuit (either "affirming" or "reversing" the trial court). These written opinions, if comparable in the underlying facts, are binding upon future cases within that state. 

#### Appellate Courts Affirm or Reverse Trial Courts' Rulings on Motions for Summary Judgement

This project will provide predictive value on a single component of a lawsuit: a motion for summary judgment. In civil (non-criminal) cases, a motion for summary judgment (or "MSJ") may be successful if, when the facts are assumed true in the other side's favor, the side bringing the motion would still be entitled to judgment as a matter of law. For instance, in a contract formation case, if the facts under the opponent's best world simply don't amount to the legal standard for formation of a contract, an MSJ may be granted. 

MSJs are significant because if a court granted an MSJ, that would typically end a case. This usually results in an appeal (asking a higher court to confirm whether the lower court made the right decision). Every litigant is entitled to a first-level appeal in North Carolina. Accordingly, the body of North Carolina appellate opinions will address many cases where MSJs have been allowed, providing some insight into the features common to those cases where the decisions on these motions have either been affirmed or reversed.

#### A Predictor for a Given Set of Features Will Assist Stakeholders 

Like most components of legal representation, MSJs can be expensive, costing from a few thousand dollars in a lower-value case, to many multiples of this as the stakes go up. While attorneys' and law firms' judgment on meeting the legal standard is crucial and fact-dependent, their collective experience with a given judge or on a given issue can also be valuable; however, an objective prediction based upon thousands of cases will valuably assist stakeholders in performing an informed and accurate cost-benefit analysis. 

### Process

#### Presumptions/Considerations

This model returns a relative probability of success in the trial court, inferred from appellate opinions (North Carolina does not provide access to trial court rulings, which actually lack any insightful details anyway). This inference may be affected by such factors as the typical deference shown by courts of appeals; so, it should be clear that the model provides a __relative__ probability (as compared to the probability of an average judge/county/etc). Moreover, it presumes that __every__ granted MSJ is appealed (which is likely an overestimation), and that the legal bases for summary judgment (as set forth in N.C.G.S. s. 1A-1, Rule 56) are met, or are at least arguable.  

#### Creating the Data Set

Opinions are stored in individual PDF files on the North Carolina Court of Appeals' web site. Beautiful Soup was used to generate a list of web addresses, which were then gathered via requests. PDFs were imported, converted to strings using the PDFMiner library, combined into an initial Pandas DataFrame, and then filtered to only those opinions containing the phrase "summary judgment". The dataset was further reduced to minimize or eliminate other complicating factors. 

Labels were extracted using regular expressions with decreasing levels of confidence. Then, feature sets were created, including the trial judge, case type, and county. Some of these were simple, while others required more detailed extractions, such as the case-type extraction. This function was built using a keyword dictionary, with the most prominent case-type match resulting. The features were reassuring; for instance, the following chart illustrates that the proportion of decisions affirming/reversing summary judgment does **not** overtly follow any time-trends (suggesting the historical performance does not need to be adjusted):

<img src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/motions_by_year.png">

On other hand, but equally re-affirming, the case-type predictor shows different relative degrees of probablity of being affirmed, as shown below:

<img align="center" src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/motions_by_case_type.png">

Similarly, the trial judges also have varying degrees of likelihood of being affirmed or reversed. The following illustrates the top fifteen judges (based upon the number of their cases involving summary judgment motions), ranked by their percentage of cases being affirmed.  (_This is a retrospective review; Judge Spainhour -- who granted one of my summary judgment motions -- passed away in 2020, and Judge Eagles was appointed as a federal district court judge in 2010, while this model only involves state-court cases._)  

<img align="center" src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/top_ten_judges.png">

### Modeling + Results

Experiments were run with several different classifiers, starting with logistic regression and then adding a simple neural network, XG Boost, and random forest models. Ultimately, a stacking classifier was also used, combining all of these models together. The data was imbalanced between the two classes (roughly 3:1), so the data were run through the models both as unbalanced and balanced using Smote-NC. When running the neural network on balanced data, the model returned similar loss curves between training and validation sets (4 layers, 3 with dropouts); however, the loss curves kept diverging with the unbalanced data.  This was a blessing in disguise; when the neural network was left out of the stacking classifier and ran with the unbalanced data, the stacking classifier (running XG Boost and random forest, with logistic regression as the final estimator) demonstrated the best performance so far (accuracy of 0.76 and f1 score of 0.86).  

In the final Streamlit app, when a judge, county, and case-type are selected, the model will provide and plot the relative likelihood of a summary judgment motion being affirmed, as illustrated below:
<p  align="center" >
<img src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/Streamlit_Selection.jpg?raw=true" width=400>
<img src="https://github.com/jnels13/NC_Litigation_Predictor/blob/main/images/Streamlit_Predict.jpg?raw=true" width=400>
</p>
The probability of all cases to be affirmed is rather high -- generally in the 70%'s -- though the model is more intended to demonstrate where a given case lies in relation to the mean likelihood of being affirmed.

### Upcoming Work

Upcoming work is as follows: <ul>
    <li> Create pipeline to retrain model as new opinions come out (currently current through end of 2020).
</ul>
