# Introduction
Hello there! 👋 Thanks for checking out my project!

During my learning journey I decided to get use of my Python skills and to bring to life this **data analysis mini-project**. 📊

Since I live in Poland and it's May 2025, the election campaign has just ended and we have just participated in the first round of presidential election. I thought it might be a good material to work on. My objective was to deliver an analysis that hasn't been seen before — something unusual, maybe even odd. 😎

This is how this crazy idea came to my mind:

*"What if we compare the election results from the first round with some absurd statistical data and check if there is a correlation?"*

Buckle up, let's start! 🎬

# Some theory

Before we dive into the analysis, let's focus on the methodology I applied here.

### Data source

All data I used in this project comes from two sources:

1. [wybory.gov.pl](https://www.wybory.gov.pl/) — the official site of the Polish government with the election results
2. [Bank Danych Lokalnych (GUS)](https://bdl.stat.gov.pl/bdl/start) — the database of Central Statistical Office (Główny Urząd Statystyczny — GUS)

I used these websites to access and download the data I was interested in and save them in the CSV format locally on my machine.

I downloaded two datasets from gov.pl: wybory_2025.csv and wybory_2020.csv, both being the election results of the first round. 📩

From GUS I get another three files: biblioteki.csv (data on public libraries), kultura.csv (data on cultural centres and their activities) and finally kosze.csv (data on... the number of trash bins). 🚮

All datasets are aggregated at the county level (pl. *powiat*). There are 380 counties in Poland.

### Tools used

In this project I focused on **Python**, which is why all analysis was done in my **Jupyter Notebook** file. I used **VS Code** to write and edit my code. 🐍

And yes, I'm ashamed to admit it: I also used **AI** (ChatGPT and GitHub Copilot) which was actually very helpful. This way, I could learn even more by interacting with the chatbot and deep diving into particular questions. 🖥️

If you want to follow my analysis more in detail, I recommend you heading over to my [Jupyter Notebook file](/wybory.ipynb) where you can find the whole script.

### A few words about statistics

Finally, I used some basic statistical concepts in my analysis. 📈

A key concept here is the **correlation**. 🧠

Correlation is a statistical measure that shows how strongly two variables move together. If they increase or decrease together, the correlation is positive. If one increases while the other decreases, it's negative. If they don’t affect each other, the correlation is close to zero.

The most common measure is Pearson’s correlation coefficient, which ranges from -1 to 1, where:
- 1 = perfect positive correlation
- 0 = no linear correlation
- -1 = perfect negative correlation

![Correlation equation](/Pictures/Correlation%20equation.png)

Where:
- x i, y i = each individual value of variables X and Y
- x, y = the average (mean) of X and Y
- ∑ = sum across all data points

One important thing to keep in mind is the fact that the correlation tells us **how strong** and in **what direction** the relationship is but **NOT why** it happens.

# The analysis

## Data Cleaning

To perform the analysis I merged all files into one using the territorial code (column 'kod_teryt') as key:

``` python
# merging the additional datasets with the main dataframe
dfmerged = dfmerged.merge(kosze, left_on='kod_teryt', right_on=['kod'], how='left')
dfmerged = dfmerged.merge(biblioteki, left_on='kod_teryt', right_on=['kod'], how='left')
dfmerged = dfmerged.merge(kultura, left_on='kod_teryt', right_on=['kod'], how='left')
```

I did a simple data cleaning by converting some columns to the right data type (float): 🧹

``` python
# Cleaning up the merged dataframe by converting specific columns to numeric types
dfmerged['liczba_koszy_na_100_mieszkancow'] = dfmerged['liczba_koszy_na_100_mieszkancow'].astype(str).str.replace(',', '.').astype(float)
dfmerged['biblioteki_1000_ludnosci'] = dfmerged['biblioteki_1000_ludnosci'].astype(str).str.replace(',', '.').astype(float)
dfmerged['wypozyczenia_na_czytelnika'] = dfmerged['wypozyczenia_na_czytelnika'].astype(str).str.replace(',', '.').astype(float)
dfmerged['udzial_osob_zaangazowanych_pow_60_roku'] = dfmerged['udzial_osob_zaangazowanych_pow_60_roku'].astype(str).str.replace(',', '.').astype(float)
dfmerged['czlonkowie_grup_artystycznych_na_1000_ludnosci'] = dfmerged['czlonkowie_grup_artystycznych_na_1000_ludnosci'].astype(str).str.replace(',', '.').astype(float)
```

## 1. How stable is the electorate of PiS?

Two of the candidates starting this year were eligible also during the last election five years ago: Rafał TRZASKOWSKI and Szymon HOŁOWNIA.

However, people tend to vote rather for the party behind a candidate, not just for the candidate. Or do they? 🤔

Let's find out! 😎

The party Law and Justice (Prawo i Sprawiedliwość — PIS) has been one of the two strongest parties in Poland for the last two decades. This year the candidate supported by PiS is Karol NAWROCKI. In 2020 it was Andrzej DUDA who actually won the election and became president. 🖊️

Based on the election results I plotted the results in percentage of these two candidates using a scatter plot. Then, I calculated the correlation coefficient.

![Duda-Nawrocki](/Pictures/Duda%20Nawrocki.png)

Here is the code I used to generate this graph:

``` python
import matplotlib.pyplot as plt
x_axis = 'DUDA_2020'
y_axis = 'NAWROCKI_2025'

dfmerged.plot(kind='scatter', x=x_axis, y=y_axis)
plt.show()

correlation = dfmerged[[x_axis, y_axis]].corr().iloc[0, 1]
print(f"Correlation coefficient: {correlation:.4f}")
```

As we can see, the correlation is very strong which is also represented visually on the scatter plot. Out of curiosity, I decided to compare it with the correlation between the election results of TRZASKOWSKI in 2020 and 2025.

I adjusted the code by changing the variables *x_axis* and *y_axis*. The result was a correlation coefficient at the level of 0.9868.

It seems to really confirm the hypothesis that people tend to vote for the party, not just for the candidate. We have seen that the correlation between the number of votes for two candidates from the same party is as strong (or even slightly stronger) than between the results of the same candidate over the last 5 years. 🗳️

## 2. Where did Nawrocki and Trzaskowski perform better/worse than Duda and Trzaskowski in 2020? (voters retention)

Let's now investigate the top 5 and bottom 5 counties (powiats) where NAWROCKI performed better/worse than DUDA and where TRZASKOWSKI performed better/worse than 5 years ago. 📉

I used the following logic to obtain 4 tables (2 for each candidate: top 5 improvement and bottom 5):

``` python
nawrocki_top10_diff = dfmerged.nlargest(5, 'NAWROCKI_VS_DUDA_DIFF')[[
    'powiat_2025', 'TRZASKOWSKI_2020', 'TRZASKOWSKI_2025', 'TRZASKOWSKI_DIFF',
    'DUDA_2020', 'NAWROCKI_2025', 'NAWROCKI_VS_DUDA_DIFF'
]].round(2)
```
### Nawrocki — top 5
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>powiat_2025</th>
      <th>TRZASKOWSKI_2020</th>
      <th>TRZASKOWSKI_2025</th>
      <th>TRZASKOWSKI_DIFF</th>
      <th>DUDA_2020</th>
      <th>NAWROCKI_2025</th>
      <th>NAWROCKI_VS_DUDA_DIFF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>254</th>
      <td>Sopot</td>
      <td>0.53</td>
      <td>0.50</td>
      <td>-0.03</td>
      <td>0.23</td>
      <td>0.17</td>
      <td>-0.06</td>
    </tr>
    <tr>
      <th>90</th>
      <td>Zielona Góra</td>
      <td>0.46</td>
      <td>0.48</td>
      <td>0.02</td>
      <td>0.26</td>
      <td>0.18</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>252</th>
      <td>Gdynia</td>
      <td>0.48</td>
      <td>0.47</td>
      <td>-0.01</td>
      <td>0.25</td>
      <td>0.17</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>251</th>
      <td>Gdańsk</td>
      <td>0.47</td>
      <td>0.43</td>
      <td>-0.04</td>
      <td>0.25</td>
      <td>0.17</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Poznań</td>
      <td>0.47</td>
      <td>0.44</td>
      <td>-0.03</td>
      <td>0.23</td>
      <td>0.15</td>
      <td>-0.08</td>
    </tr>
  </tbody>
</table>
</div>

### Nawrocki — bottom 5
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>powiat_2025</th>
      <th>TRZASKOWSKI_2020</th>
      <th>TRZASKOWSKI_2025</th>
      <th>TRZASKOWSKI_DIFF</th>
      <th>DUDA_2020</th>
      <th>NAWROCKI_2025</th>
      <th>NAWROCKI_VS_DUDA_DIFF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>162</th>
      <td>siedlecki</td>
      <td>0.10</td>
      <td>0.10</td>
      <td>0.00</td>
      <td>0.69</td>
      <td>0.47</td>
      <td>-0.22</td>
    </tr>
    <tr>
      <th>55</th>
      <td>chełmski</td>
      <td>0.14</td>
      <td>0.15</td>
      <td>0.01</td>
      <td>0.63</td>
      <td>0.41</td>
      <td>-0.22</td>
    </tr>
    <tr>
      <th>128</th>
      <td>proszowicki</td>
      <td>0.13</td>
      <td>0.16</td>
      <td>0.02</td>
      <td>0.64</td>
      <td>0.43</td>
      <td>-0.21</td>
    </tr>
    <tr>
      <th>151</th>
      <td>ostrołęcki</td>
      <td>0.12</td>
      <td>0.14</td>
      <td>0.02</td>
      <td>0.66</td>
      <td>0.44</td>
      <td>-0.21</td>
    </tr>
    <tr>
      <th>206</th>
      <td>przeworski</td>
      <td>0.12</td>
      <td>0.14</td>
      <td>0.02</td>
      <td>0.67</td>
      <td>0.46</td>
      <td>-0.21</td>
    </tr>
  </tbody>
</table>
</div>

### Trzaskowski — top 5
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>powiat_2025</th>
      <th>TRZASKOWSKI_2020</th>
      <th>TRZASKOWSKI_2025</th>
      <th>TRZASKOWSKI_DIFF</th>
      <th>DUDA_2020</th>
      <th>NAWROCKI_2025</th>
      <th>NAWROCKI_VS_DUDA_DIFF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>78</th>
      <td>krośnieński</td>
      <td>0.33</td>
      <td>0.41</td>
      <td>0.07</td>
      <td>0.34</td>
      <td>0.24</td>
      <td>-0.10</td>
    </tr>
    <tr>
      <th>185</th>
      <td>krapkowicki</td>
      <td>0.31</td>
      <td>0.38</td>
      <td>0.07</td>
      <td>0.35</td>
      <td>0.20</td>
      <td>-0.15</td>
    </tr>
    <tr>
      <th>87</th>
      <td>żarski</td>
      <td>0.34</td>
      <td>0.40</td>
      <td>0.06</td>
      <td>0.35</td>
      <td>0.24</td>
      <td>-0.11</td>
    </tr>
    <tr>
      <th>222</th>
      <td>hajnowski</td>
      <td>0.28</td>
      <td>0.33</td>
      <td>0.06</td>
      <td>0.31</td>
      <td>0.19</td>
      <td>-0.12</td>
    </tr>
    <tr>
      <th>191</th>
      <td>strzelecki</td>
      <td>0.30</td>
      <td>0.35</td>
      <td>0.06</td>
      <td>0.40</td>
      <td>0.23</td>
      <td>-0.17</td>
    </tr>
  </tbody>
</table>
</div>

### Trzaskowski — bottom 5
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>powiat_2025</th>
      <th>TRZASKOWSKI_2020</th>
      <th>TRZASKOWSKI_2025</th>
      <th>TRZASKOWSKI_DIFF</th>
      <th>DUDA_2020</th>
      <th>NAWROCKI_2025</th>
      <th>NAWROCKI_VS_DUDA_DIFF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>178</th>
      <td>Warszawa</td>
      <td>0.48</td>
      <td>0.42</td>
      <td>-0.05</td>
      <td>0.27</td>
      <td>0.18</td>
      <td>-0.09</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Wrocław</td>
      <td>0.44</td>
      <td>0.40</td>
      <td>-0.04</td>
      <td>0.28</td>
      <td>0.18</td>
      <td>-0.10</td>
    </tr>
    <tr>
      <th>251</th>
      <td>Gdańsk</td>
      <td>0.47</td>
      <td>0.43</td>
      <td>-0.04</td>
      <td>0.25</td>
      <td>0.17</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Poznań</td>
      <td>0.47</td>
      <td>0.44</td>
      <td>-0.03</td>
      <td>0.23</td>
      <td>0.15</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>134</th>
      <td>Kraków</td>
      <td>0.38</td>
      <td>0.35</td>
      <td>-0.03</td>
      <td>0.32</td>
      <td>0.21</td>
      <td>-0.11</td>
    </tr>
  </tbody>
</table>
</div>

This leads to interesting conclusions:

- **In all counties Nawrocki actually performed worse than Duda.** This seems to make sense as in this year's election two other right-wing candidates (BRAUN and MENTZEN) achieved much better results than their colleague BOSAK in 2020. The voters could have switched from one right-wing party to another.

- **Nawrocki had the best voters retention in Tricity (Trójmiasto: Gdańsk, Gdynia, Sopot).** This is quite surprising. Rafał TRZASKOWSKI used excatly the opposite claim as an argument against Karol NAWROCKI. Referring to a recent appartment scandal related to NAWROCKI, TRZASKOWSKI said that "people were moved by this event, especially in Tricity, what can be seen by looking at the results". Well, the data proves that it actually isn't true!

- **Trzaskowski lost the most voters in the biggest cities including Warsaw.** This one is also surprising. TRZASKOWSKI, being the mayor of Warsaw, was expected to perform very well there as well as in other big cities where he is seen as the strongest opposition to NAWROCKI. One possible explanation of this fact is stronger differentiation amongst the voters which allowed the candidates other than the top 2 to gather significantly more votes. This way they might have "stolen" these votes from TRZASKOWSKI as we have just seen it happen to NAWROCKI.

## Public libraries and voters' behaviour

Let us now move to a bit more specific comparisons and see if the voters' behaviour is somehow related to the number of readers in the public libraries. 📶

Here, once again, I used a scatter plot and calculated the correlation coefficient to investigate the trends.

The results? Not really surprising, but we can observe a very light correlation. 📖

The strongest (but still quite small) **positive correlation** was noticed in case of **Adrian ZANDBERG**, the left-wing candidate:

![ZANDBERG-libraries](/Pictures/Biblioteki%20Zandberg.png)

The strongest (but even smaller than in case of ZANDBERG) **negtive correlation** was noticed for **Sławomir MENTZEN**, the right-wing candidate:

![MENTZEN-libraries](/Pictures/Biblioteki%20Mentzen.png)

It is also worth mentioning that in case of other candidates the correlation was smaller, nevertheless there was a tendency there too: all right-wing candidates had a negative correlation wheras the centre and left-wing — a positive one.

These findings don't prove anything yet but surely can't be ignored too.

There is also another conlcusion: reading and actively voting are somehow related: 📚

![frequence-libraries](/Pictures/Biblioteki%20Frekwencja.png)

## Culture centres and their activity

In the next step I checked the relationship between the number of votes for a specific candidate and the activities of culture centres. 🎭

The correlation turned out to be pretty small once again but there is also an interesting pattern hidden out there: all candidates from the right-wing have a positive correlation with the number of number of artistic group members per 1000 inhabitants: **the more artistic group members, the more votes for BRAUN, MENTZEN or NAWROCKI in a given county**. On the other hand, in case of all candidates from the opposite side a negative relationship can be seen.

Below is the scatter plot showing the correlation for BRAUN who has the strongest correlation:

![Braun-culture](/Pictures/Kultura%20Braun.png)

And here are all "main" candidates:

![Culture-comparison](/Pictures/Kultura%20razem.png)

## And finally — trash bins!

Yes, trash bins! 🗑️

Let's see if there is any correlation between the number of trash bins per 100 inhabitants and the votes for the specific candidates.

![Trash-bins](/Pictures/Kosze.png)

Well...

Actually, there is no correlation here! 😂

# Conclusion

To be honest, I did't expect to find so many interesting insights.

One final conclusion? 🤔

A data analyst needs to be prepared for everything — even the greatest surprise. And, of course, they shoud never just try to prove their suppositions. Ever❗

### Who am I?

I am Marcin Czerkas — a data analyst who hasn't paid a single coin for my analytical education and yet learns every day on my own, using free learning resources. 🎓

I work with data since 2023. At that time I changed job and started helping my new team by automating the process and building dashboards. Currently, I work in the asset risk department and help my company not to lose money by spotting dangerous trends in the data early on. ⚡

When I transitioned into the world of data, I started a learning journey that lasts until now and is probably going to last for a long time. Projects like this one help me test my analytical skills in practice and showcase them.

### 🔗 **Visit [my LinkedIn profile](https://www.linkedin.com/in/marcin-czerkas-95150727a/) to learn more.**