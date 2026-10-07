# Netflix Subscriber Churn and Engagement Analysis

An analysis of 50,000 streaming subscribers, with one row per user and 20 fields covering demographics, plan, viewing behavior and whether the user churned. About 19.9% of users have churned, which is 9,964 people and $122K of the $616K in monthly recurring revenue. I wanted to know who is churning and why, and whether any pattern in the data is strong enough to base a retention plan on.

My answer: nothing here explains churn. Every chart I built shows small differences between groups, and none of them hold up once I account for how much a gap of that size can vary by chance. A model fed every field at once predicts churn no better than a coin flip. So I wouldn't spend retention budget on targeting by age, country or genre. I'd run a controlled test and collect better data first.

Analysis was done in Python (pandas, SciPy, scikit-learn) and the charts were built in Tableau.

## Data

The dataset is synthetic, not real Netflix data. Almost every field is spread evenly and has no link to churn, which is typical of generated data, so the null result describes this dataset and may not carry over to a real subscriber base. The part that transfers is the method.

I checked it before using it. There are no duplicate users and no missing values, and the ranges are sensible (ages 18 to 64, account age 1 to 59 months). Two oddities matter later. The monthly fee has three values (7.99, 12.99, 15.99) but is unrelated to the subscription plan, so a Premium user is as likely to pay 7.99 as 15.99. And there is no signup date, only account age in months. Revenue in this README means the sum of the monthly fee.

## Churn is about 20% in every plan and age group

![Churn rate by plan and age bin](demographic-vs-loyalty.png)

By plan, churn runs from 19.2% for Basic to 20.4% for Premium, with Standard at 20.1%. The heatmap breaks it down by age, and its cells range from 18.0% to 21.4%. The darkest and reddest cells look like segments worth attention, but they are mostly the smallest ones. The age 10 to 19 group has only 618 to 824 users per cell, against over 4,000 in the larger cells, and small groups swing more. When I shuffled the churn labels across users at random, the spread between the highest and lowest cell came out about the same size as the real one.

The color scale also runs only from 18% to 21%, so a gap of a couple of points looks dramatic on the chart. I'd read this as one flat churn rate of about 20%.

## Country differences are within the margin of error

![Churn rate by country](global-churn.png)

Churn goes from 19.2% in the USA to 21.0% in Australia, with Japan at 20.8% and India at 20.3%. Each country has around 5,000 users, which gives each rate a margin of error of about 1.1 points. That covers most of the range. Australia is the closest to a real signal at 21.0% against 19.8% for everyone else, but it falls short of being conclusive. I'd keep an eye on it and not build a market plan around it.

## Revenue lost by genre is flat because genres are the same size

![Monthly revenue lost to churn by favorite genre](genre-financial-impact.png)

Monthly revenue lost to churn is between $14.7K for Drama and $15.7K for Horror, and each genre holds about 12% of the total. This chart mostly shows how many users each genre has. All eight genres have roughly 6,200 users and churn rates between 19.3% and 20.5%, so the boxes look alike. Churned users also pay almost exactly what active users pay ($12.28 against $12.33 a month), so lost revenue follows lost users one for one. Nothing here supports changing content or pricing for one genre.

## The segmentation describes users and does not predict risk

![Users by lifecycle segment](user-segmentation.png)

I split users into four lifecycle segments. Super Fans are active users watching more than 60 minutes on average, at 33,025 users or 66%. Lost Customers are the 9,964 who churned. At Risk users are active but watch 60 minutes or less and haven't logged in for more than 14 days, at 5,201 users holding $64K of monthly revenue. Active Users are the remaining 1,810.

Two issues here. Super Fan is too easy to qualify for: 60 minutes sits at about the 18th percentile of watch time, so most active users clear it. And At Risk is not a proven risk score. Applying the same profile to all users, 6,545 match it and they churn at 20.5%, against 19.8% for everyone else. That gap is too small to be useful. Still, it is a reasonable audience to try a retention offer on, as long as the result is measured against a control group.

## No engagement field separates churned users from active ones

I compared churned and active users on every numeric field, including watch time, sessions per week, completion rate, ratings, recommendation clicks and days since last login. The averages match within about 1%. I also fit a logistic regression and a gradient boosting model on all fields together, and both scored a cross-validated AUC of about 0.50, which is chance level. With 50,000 users, a real effect of one or two points would have shown up, so any true driver here is small or sits in data this set doesn't have.

## The January dip is an artifact

![Registered users by calendar month and plan](user-acquisition-growth-trend.png)

The chart shows registrations dropping sharply in January, then staying flat all year. It is not a real dip. The dataset has no signup date, so I rebuilt one by counting back from each user's account age in months. Account age runs from 1 to 59, and 59 is not a multiple of 12, so one calendar month ends up with four cohorts and the other eleven get five. That month has 3,378 users against an average of about 4,240 for the others, which is the 4 to 5 ratio the construction predicts. Outside that month the line is flat, so this field can't show acquisition growth or seasonality at all.

## What I would do with this

1. Don't fund retention targeted by age, country or genre on this data. None of those splits showed a real difference.
2. Run a randomized retention test on the At Risk audience, with a holdout group, and measure how much extra retention the offer buys. Each point of churn is worth about $6.2K a month, or about $74K a year.
3. Collect the fields that could actually explain churn: cancellation reasons, billing failures, price changes, support contacts and an exact churn date.
4. Rebuild the segments on percentile thresholds and check them against churn before using them again.
5. Fix the data definitions, starting with a real signup date and a fee that matches the plan.

## Limitations

The data is synthetic, so a null result here may reflect how it was generated. The churn flag has no date or time window, which rules out survival analysis, cohort retention and time-to-churn. Because fee and plan are unrelated, I would not draw conclusions about revenue by plan. Finding no difference doesn't prove none exists, only that anything real is smaller than about one or two points. And this is observational data, so it shows association and not cause. A retention experiment is what would establish cause.
