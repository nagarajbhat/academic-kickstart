---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Predicting Titanic movie's character's survival chance using ML"
subtitle: ""
summary: "We try to predict some of the titanic movie's character's survival chances using the famous titanic Kaggle dataset"
authors:
- admin
tags: ["python","Data science","Machine learning","kaggle"]
categories: ["Data Stories"]
date: 2020-05-21T22:46:35+10:00
lastmod: 2020-05-21T22:46:35+10:00
featured: true
draft: false

type: post 
comments: true 
share: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  placement: 3
  caption: ""
  focal_point: ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

The [titanic dataset](https://www.kaggle.com/c/titanic) must be one of the most overused one, visited by most of the data scientist's as a starting point in Kaggle.
So you might wonder what is the need to write a blog about something so familiar?

I picked this dataset for a couple of reasons,
Firstly, I was curious to compare the movie character's survival with the actual prediction.
And another reason was to see if some of the feature engineering techniques I learned recently improves the model.

**My top model got an accuracy of 80.382%, ranking in top 9% in Kaggle.** Of course, there was some room for improvement, especially in the feature engineering part.

![image](/img/predicting-titanic-survival/rank.PNG)
*Fig: Current model ranking*

So Can we predict If Jack survived or not? 

![image](/img/predicting-titanic-survival/rose_j_meme1.jpg)
*Fig: Jack-rose meme* - [credits](https://imgflip.com/i/2m0597)

You might have seen dozens of memes like the one above. But we are not going to predict whether jack could fit on the wooden door or not.
Rather we will take a look at Jack's information even before he made into the Titanic ship and predict his and other character's survival chances.


Let us take a look at the titanic dataset and the features given to us.

![image](/img/predicting-titanic-survival/dataset.PNG)
*Fig: Jack's survival prediction*


The columns given to us are -  Survived, Pclass, Name, Sex, Age, SibSp, Parch, Ticket, Fare, Cabin, Embarked. Ticket number and Name don't make much of a difference hence we will drop the features. There are too many missing values in Cabin, so it is more noise than data, hence we will drop this feature too.

There are few features which do contribute to the prediction but for this blog, where we want to find survival chances of movie characters,
it will be difficult to find their values, hence we will drop them too, they are - SibSp(has sibling, spouse), Parch(has parent, child), Fare.

We will discuss later on how to make use of important features, and do feature engineering to increase model performance. But for now, we are left with 4 features - Pclass(Passenger class), Sex, Age, Embarked (the location where the passenger boarded the ship). Embarked has 3 class - C = Cherbourg, Q = Queenstown, S = Southampton.
 **After reducing the features my model performance dropped to 75.11%.**

Using these features I created a pipeline with column transformation to handle categorical and numerical features, and also the model (XGBoost).

So I extracted most of the character information from fandom website - [link](https://jamescameronstitanic.fandom.com/wiki/James_Cameron%27s_Titanic_Wiki)

### 1. Jack Dawson

Jack was a 20-year-old artist who managed to get into titanic by winning a game of poker. He was the love interest of Rose and boarded the ship at Southampton on a third-class ticket.
We will enter these details in the format shown below 

[ 'Pclass', 'Sex' , 'Age' , 'Embarked' ]

 and predict the survival . **1 means survived, 0 means did not survive.**

![image](/img/predicting-titanic-survival/predict_jack.PNG)
*Fig: Jack's survival prediction*

As seen above, after we enter the details of Jack, the output array([0]) indicates Jack would not have survived.

![image](/img/predicting-titanic-survival/rose_j_meme4.jpg)
*Fig: Jack's meme*

### 2. Rose DeWitt Bukater
Rose was an American socialite. She was 17 years old, boarded the ship at Southampton on a first-class ticket.
She was engaged to Cal. Due to her lifestyle and treatment she received, she tried to commit suicide, until she was saved by Jack, whom she developed a love interest with.

![image](/img/predicting-titanic-survival/predict_rose.PNG)
*Fig: Rose's survival prediction*

According to the prediction (array([1])) rose managed to survive, which holds good with the movie.
After surviving, rose dumped Cal, pursued her dreams, became a successful actress, even flew a plane, and lived for more than a 100 years.

![image](/img/predicting-titanic-survival/rose_j_meme5.jpg)
*Fig: Rose's meme*

### 3. Caledon Nathan Hockley

Also known as "Cal". He was an American Industrialist and the heir to a Pittsburgh steel fortune. He was Rose's fiancee, 30 years of age, and boarded the ship at Southampton on a first-class ticket.

![image](/img/predicting-titanic-survival/predict_cal.PNG)
*Fig: Cal's survival prediction*

According to the model cal dies in the Titanic disaster, but in the movie he manages to survive.

Cal's money didn't save him, although he eventually escaped when he found a lost child and claimed that child to be his own.
![image](/img/predicting-titanic-survival/cal_meme.jpg)
*Fig: Cal's meme*


### 4. predicting other characters  

Let us predict for a few other characters.
![image](/img/predicting-titanic-survival/predict_all.PNG)
*Fig: predict for other characters*

According to prediction, Ruth, Margret and Helga survive and Spicer, Fabrizio, and Thomas died.
The prediction is correct for all except Helga Dahl, who happened to die.



### The gender-class pattern
After going through some of these characters the pattern is becoming increasingly clear, isn't it?

![image](/img/predicting-titanic-survival/pattern.PNG)
*Fig: Gender-class Pattern*

If you are a female and have taken a first-class ticket, there's an extremely high chance of survival. Whereas if you are a male and have taken a third-class ticket, you are pretty much screwed!

Of course we know of the "women and children" first policy which was deployed during evacuation and filling up lifeboat.

Looking at the predictions it might seem that female characters have all survived and male characters have died.
But this is not true when you consider the real data, to illustrate I will take a couple of hypothetical characters,
where the female dies and the male survives.

![image](/img/predicting-titanic-survival/predict_hyp.PNG)
*Fig: predict for hypothetical characters*

Men who have taken first-class tickets and are around the age of 33-37 have better survival chances than other men.


### Age class pattern

I divided the people into 7 age brackets, to see how they survived.

![image](/img/predicting-titanic-survival/age_class.PNG)
*Fig: Age class Pattern*

Bracket 0(children below 11) and Bracket 1 (11 to 17 yrs) had the highest survival rate, this too can be attributed to "women and children" first policy. 

Bracket 2(17 to 22yrs) and Bracket 7 (above 60yrs) had the worst survival rate


## **Improving the model**

### Things that worked for me

- Hyperparameter tuning of XGBoost classifier (learning rate, max_child, and n_estimators)
- Considering other features such as fare, SibSp, and Parch.
- imputing numerical features using the median, and categorical features using the most frequent value.
- One hot encoding categorical features.

### Things that didn't make any difference 

-  Including cabin class - Since cabin class is more noise than data it is better to drop it. However I did try to extract the first letter of the cabin feature, and classify it into 6-7 cabin classes. But this did not add any value.
- Adding count encoder (for categorical features) did not make any difference to model performance.

### Things that may work (but I haven't tried it yet) 

- Combining SibSp and Parch to create a new feature - "Not Alone". This will be 1 if the combined value is above 0 indicating the person was not alone. If the combined value is 0, then the person would be alone.
- performing Grid Search for hyperparameter tuning

This is part of my #100daysofcode journey, check out the progress [here](https://github.com/nagarajbhat/100DaysOfCode)