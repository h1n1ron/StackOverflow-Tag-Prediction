# Stack Overflow: Tag Prediction

<h1>1. Business Problem </h1>

<p style='font-size:18px'><b> Description </b></p>
<p>
Stack Overflow is the largest, most trusted online community for developers to learn, share their programming knowledge, and build their careers.<br />
<br />
Stack Overflow is something which every programmer use one way or another. Each month, over 50 million developers come to Stack Overflow to learn, share their knowledge, and build their careers. It features questions and answers on a wide range of topics in computer programming. The website serves as a platform for users to ask and answer questions, and, through membership and active participation, to vote questions and answers up or down and edit questions and answers in a fashion similar to a wiki or Digg. As of April 2014 Stack Overflow has over 4,000,000 registered users, and it exceeded 10,000,000 questions in late August 2015. Based on the type of tags assigned to questions, the top eight most discussed topics on the site are: Java, JavaScript, C#, PHP, Android, jQuery, Python and HTML.<br />
<br />
</p>

<p style='font-size:18px'><b> Problem Statemtent </b></p>
Suggest the tags based on the content that was there in the question posted on Stackoverflow.

<p style='font-size:18px'><b> Source:  </b> https://www.kaggle.com/c/facebook-recruiting-iii-keyword-extraction/</p>

<h2> 1.2 Source / useful links </h2>

Data Source : https://www.kaggle.com/c/facebook-recruiting-iii-keyword-extraction/data <br>
Youtube : https://youtu.be/nNDqbUhtIRg <br>
Research paper : https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tagging-1.pdf <br>
Research paper : https://dl.acm.org/citation.cfm?id=2660970&dl=ACM&coll=DL

<h2>1.3 Mapping the real-world problem to a Machine Learning Problem </h2>
<h3> 1.3.1 Type of Machine Learning Problem </h3>

<p> It is a multi-label classification problem  <br>
<b>Multi-label Classification</b>: Multilabel classification assigns to each sample a set of target labels. This can be thought as predicting properties of a data-point that are not mutually exclusive, such as topics that are relevant for a document. A question on Stackoverflow might be about any of C, Pointers, FileIO and/or memory-management at the same time or none of these. <br>
__Credit__: http://scikit-learn.org/stable/modules/multiclass.html
</p>

<h3>1.3.3 Performance metric </h3>
<b>Micro-Averaged F1-Score (Mean F Score) </b>: 
The F1 score can be interpreted as a weighted average of the precision and recall, where an F1 score reaches its best value at 1 and worst score at 0. The relative contribution of precision and recall to the F1 score are equal. The formula for the F1 score is:

<i>F1 = 2 * (precision * recall) / (precision + recall)</i><br>

In the multi-class and multi-label case, this is the weighted average of the F1 score of each class. <br>

<b>'Micro f1 score': </b><br>
Calculate metrics globally by counting the total true positives, false negatives and false positives. This is a better metric when we have class imbalance.
<br>

<b>'Macro f1 score': </b><br>
Calculate metrics for each label, and find their unweighted mean. This does not take label imbalance into account.
<br>

https://www.kaggle.com/wiki/MeanFScore <br>
http://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html <br>
<br>
<b> Hamming loss </b>: The Hamming loss is the fraction of labels that are incorrectly predicted. <br>
https://www.kaggle.com/wiki/HammingLoss <br>



<h1> 2. Appraching the problem </h1>
<h2> 2.1 EDA </h2>
<h3> 2.1.1 Data Loading and Deduplication </h3>
Pandas and Sqlite are used to load the data where 30% of total questions are duplicates. So a new database was created with no duplicates.

<h3> 2.1.2 Analysis of Tags </h3>
<ol>
  <li> First tags were tokenized</li>
  <li> Number of times a tag appeared was counted. Note that, given a question, a specific tag can appear only once.</li>
  <li> To get a clear picture . distribution of tags was observed. </li>
  <li> Number of tags per question. </li>
  <li> Wrod Cloud visualisation to get an idea of most frequent tags </li>
</ol>

<h3> 2.1.3 Data Preprocessing </h3>
<ol> 
    <li> Sample 1M data points </li>
    <li> Separate out code-snippets from Body </li>
    <li> Remove Spcial characters from Question title and description (not in code)</li>
    <li> Remove stop words (Except 'C') </li>
    <li> Remove HTML Tags </li>
    <li> Convert all the characters into small letters </li>
    <li> Use SnowballStemmer to stem the words </li>
</ol>

<h3> 2.1.4 Data Preparation </h3>
<ol>
  <li> Converting tags for multi label classification: CountVecotrizer </li>
  <li> Even after sampling 1 Million data points we have nearly 42000 tags. Now classification in One vs Rest setting with this much tags is quite cumbersome. So partial coverage method
  is used. Suppose , <a href="https://www.codecogs.com/eqnedit.php?latex=C" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C" title="C" /></a> is the set of tags. Now a subset <a href="https://www.codecogs.com/eqnedit.php?latex=C^{'}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C^{'}" title="C^{'}" /></a>, where 
  <a href="https://www.codecogs.com/eqnedit.php?latex=C^{'}\subset&space;C" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C^{'}\subset&space;C" title="C^{'}\subset C" /></a>, such that <a href="https://www.codecogs.com/eqnedit.php?latex=C^{'}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C^{'}" title="C^{'}" /></a>  partially covers
  at least 90% of the tags. One approach for partial covering is to choose most frequent tags.</li>
</ol>
  
 <h3> 2.1.4 Featurization </h3>
 Two kind of vectorizers were used for featurization:
 <ol>
  <li> TF-IDF</li>
  <li> BOW </li>
 </ol>
  
 <h3> 2.1.5 Machine Learning Models </h3>
 All models are used in One vs Rest Setting:
 <ol>
  <li> SGDClassifier with `log` loss</li>
  <li> SGDClassfier with `hinge` loss</li>
 </ol>
  
 <h3> 2.1.6 Model Performance </h3>
 
 1. TF-IDF with 1 million datapoints

  | Model | Vectorizer | Micro F1 Score | Hamming Loss|
  | --------------- | --------------- | --------------- | --------------- |
  | SGDCLassifier with log loss | TF-IDF | 0.37427 | 0.00041 |
  
 2. TF-IDF with 1 million datapoints with more weight to title and 500 tags only as 500 tags covers 90% of questions
 
  | Model | Vectorizer | Micro F1 Score | Hamming Loss|
  | --------------- | --------------- | --------------- | --------------- |
  | SGDCLassifier with log loss | TF-IDF | 0.4488 | 0.00278 |
  
 3. BOW with 4 grams and 1 million datapoints, more weight to title and 500 tags only as 500 tags covers 90% of questions
 
  | Model | Vectorizer | Alpha | Micro F1 Score | Hamming Loss |
  | --------------- | --------------- | --------------- | --------------- | --------------- |
  | SGDCLassifier with log loss | BOW | 0.001 | 0.4668 | 0.0031 |
  | SGDCLassifier with hinge loss | BOW | 0.001 | 0.4561 | 0.0032 |
 
  
 
