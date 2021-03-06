---
layout: post
title:      Hypothesis Testing - Northwind Trading Co.
date:       2019-1-31 20:44:56 +0000
permalink:  hypothesis_testing
---
>“There are two possible outcomes: if the result confirms the hypotheses, then you’ve made a measurement. If the result is contrary to the hypothesis, then you’ve made a discovery”
— Enrico Fermi 


The scientific method is a process that involves asking questions, collecting data and testing a hypothesis. This method is the basis of the scientific world and when used correctly, can be very powerful. The method consists of the six steps below: 

* Step 1: Ask a question
* Step 2: Do background research
* Step 3: Construct a hypothesis
* Step 4: Test your hypothesis by doing an experiment
* Step 5: Analyze the data and draw a conclusion
* Step 6: Share your results and iterate

In order to become well versed with the scientific method, I have completed an analysis of the Northwind Trading Co. database. I used sql to access the fictitious Microsoft database and imported the data into a jupiter notebook. This database is a schema of 13 tables that contain information relating to products, sales, customers, suppliers and employees for the fictitious company. 

After importing the data into a pandas dictionary for easier exploration I decided to approach this project as I would a business update. In my previous managerial work, I often focused on a few key aspects: People, Sales, Profit. I believe that focusing on these aspects, in that order, will lead to a successful business. With this in mind, I focused on the following questions:

1. Looking for the top performers: Does the UK or US sales team sell more product? Are men or women more productive? Who is the top Salesperson?
2. Is there a difference in the mean number of products sold in a sales categories by region?
3. Is there a difference in the number of imports and domestic products that customers purchase and the revenue they create?
4. Do discounts have a statistically significant effect on the number of products customers order? If so, at what level(s) of discount?

## Ask a Question

For each of these questions, I followed the same process. I started by creating my question and null/alternative hypotheses. I then decided which tests and significance value I would use. Doing this step early in the process before any analysis has been completed is a good way to ensure that there is no experimentor bias in the process. 

## Do Background Research
I then started importing the data into a DataFrame using a sql query. I found this to be the most straightforward because I could choose the exact columns that I wanted and use JOIN statements across the tables in a succinct manner. While these queries can look intimidating, it is a great way to ensure you get the exact information you need in order to answer your question. 

~~~
h3 = pd.read_sql_query('''SELECT c.categoryname categories, p.id as product, od.quantity quantity,
                            r.regiondescription regions, od. unitprice unit_price, od.Discount discount
                            FROM product p 
                            JOIN category c ON p.categoryId = c.Id
                            JOIN orderdetail od ON p.Id = od.productId
                            JOIN [order] o ON od.OrderId = o.Id
                            JOIN employee e ON o.EmployeeId = e.Id
                            JOIN employeeterritory et ON e.Id = et.employeeid
                            JOIN territory t ON et.territoryId = t.id
                            JOIN region r ON t.regionId = r.id
                            ;''', con)
~~~

I then took an extra step to calculate the total sales number for each product. We were given the number of units purchased, price and discount offered, but it was calculated by line item. In order to find the total sales per product, I multiplied the number of products sold by the price including discount when appropriate. I then grouped the data by the necessary columns to get the information I needed to start graphing the data and running tests. 

The first test I ran was a Levene’s test, which tests for homogeneity of variances. In other words, this test described how likely it is that the two datasets came from the same distribution. This information aided me in determining which additional tests were appropriate to run. 

## Test with an experiment

I used a variety of tests in order to analyze my data including two-tailed t-tests, Welch’s t-test and Tukey tests. For all of these tests I used a significance value of 0.05 and found an array of results. 

## Analyze the data and draw conclusions
This step is where it all comes together. While it is not the final step, this is what will inform any business decisions as well as any future necessary testing. It is important during this step to be be mindful of what the data is truly telling us when drawing conclusions, as it is easy to make false assumptions or make a decision that is not entirely correct. There is always the possibility that there are hidden influences that are muddling your results. 

The findings of my analysis are summarized below, as well as my ultimate recommendations for northwind trading co. moving forward. 

**Is the UK or US sales team sell more product? Are men or women more productive? Who is the top Sales Person?**

![Violin plot of sales by region and gender](img/H2_northwind.png)

I found that both regions and all sales members performed equally well. The two-tailed t-test comparing the mean sales of the US based sales team (1520.15) vs. the UK based sales team (1538.31) was insignificant with t=0.1257 and p=0.899. I then compared the mean sales of of males (1566.10) vs. females (1504.48) and found that also was insignificant with t=0.453 and p=0.65. After these tests I used a Tukey test to compare all nine employees with each other. There was no significance in sales amongst the employees. 

**Is there a difference in the mean number of products sold in a sales categories by region?**

![Box plot of category of products sold by region](img/H3_northwind.png)

I first used a Tukey test to compare the regions with each other. This found that the Eastern region sells more product than the other three regions, and the Western region also sells more than both the Southern and Northern regions. There is however no significant difference between the Southern and Northern regions. I ran the same test for sales and found that only the Eastern region has more in sales than the other three regions. After this I ran a test comparing sales categories with each other and found no significant results. 

**Is there a difference in the number of imports and domestic products that customers purchase and the revenue they create?**

![Violin plot of sales by region split by whether they are imported or domestic](img/H4_northwind.png)

The t-test comparing the mean number of imports sold (40.32) vs. domestic products (44.33) was significant with t=−2.897 and p=0.0038. The t-test comparing the mean sales of imports (540.59) vs. domestic products (891.64) was also significant with t=−4.101 and p=5.1699e−05. 

**Do discounts have a statistically significant effect on the number of products customers order? If so, at what level(s) of discount?**

![Violin plot of discount distribution by level of discount](img/H1_northwind.png)

Using Welch's t-test, we can see that there is a significant difference between the mean products sold when there was no discount offered (21.7) and any amount of discount offered (27.1) with t=−6.23 and p=5.65641429e−10. Using a Tukey test, we can further explore that 15% , 20%, and 25% means are significantly different from the mean products sold with no discounts. 5% discount is significantly different from 0 but 10% is not. There is no significant difference between the groups of discount levels.
 
## Final Conclusions
 
 These tests led me to these final conclusions.
 * Develop relationships with more domestic partners and push domestic suppliers
 * The Eastern Region is the strongest, followed by the Western Region in both sales and number of products sold. There is however no leading category.
 * Both regions and all sales members perform equally well
 * Discounts sell more products




