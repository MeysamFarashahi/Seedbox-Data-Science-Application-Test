# Seedbox-Data-Science-Application-Test
## 1- What is the aproximate probability distribution between the test group and the control group?
The first step is to import the libraries we need.
```
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style('whitegrid')
```
Then we should import the datasets.
```
test_data= pd.read_csv('testSamples.csv')
trans_data= pd.read_csv('transData.csv')
``` 
lets see how many users were in the sample
```
total= len(test_data)
```
total number of sample users is 59721
we will see the distribution between the test group and the control group in a chart.
```
plt.figure(figsize = (12,6))
ax = sns.countplot(x='test_group', data= test_data )
for p in ax.patches:
    percentage = '{:.1f}%'.format(100 * p.get_height()/total)
    x = p.get_x() + 2*p.get_width()/3
    x1 = p.get_x() + p.get_width()/3
    y = p.get_height()
    ax.annotate(percentage, (x, y),ha='center')
    ax.annotate(y, (x1, y),ha='center')
plt.xticks(fontsize = 12)
plt.yticks(fontsize = 12)
plt.xlabel('Test_group', fontsize = 12, weight='bold')
plt.ylabel('Count', fontsize = 12, weight='bold')
plt.title('Distribution between the test group and the control group', fontsize = 14, weight='bold')
plt.show()
```
![image](https://user-images.githubusercontent.com/65902301/178162568-34841081-8f62-4239-8c4f-fb328054b827.png)

Through the total 59721 users, 44886 were assigned to the control group and 14835 were assigned to the test group.
The probability that a user is assigned to the control group is approximately 75 percent
while the probabilety that a user is assigned to the test group is approximately 25 percent.

## 2-Is a user that must call-in to cancel more likely to generate at least 1 addition REBILL?
from this point, both datasets are needed at the same time so the two datasets will be mereged.
we will evaluate all the users even those without a transaction so a left join will be performed.
```
df = pd.merge(test_data,trans_data,on='sample_id',how='left')
```
 We want to find out the probabilty of at list 1 REBILL transaction given that the user must call-in to cancel.<br />P(transactopn_type == REBILL | test_group = 1) = P(transactopn_type == REBILL ∩ test_group = 1) / P(test_group = 1)

Number of users that were assigned in control group and performed a transaction:
```
ctrl_rebill = []
for i in range(len(df)):
    if df['test_group'][i] == 0 and df['transaction_type'][i] == 'REBILL':
        ctrl_rebill.append(df.sample_id.iloc[i])
dist_ctrl_rebill = len(set(ctrl_rebill))
```
Number of users that were assigned to test group and performed a transaction:
```
test_rebill = []
for i in range(len(df)):
    if df['test_group'][i] == 1 and df['transaction_type'][i] == 'REBILL':
        test_rebill.append(df.sample_id.iloc[i])
dist_test_rebill = len(set(test_rebill))
```
Between the **44886** users that were aasigned to the **control group**, **941** users generated at least 1 REBILL which is almost **2%** of them.<br />
Between **14835** users that were assigned to the **test group**, **1556** users generated at least 1 REBILL this is almost **10.5%** of them.<br />
As a result a user that must casll-in is more likely to generate REBILL

## 3- Is a user that must call-in to cancel more likely to generate more revenues?
the 'total_trans_df' show how much revenue does a unique person generates.
```
total_trans_df = (df.drop('transaction_id', axis =1)).groupby(['sample_id', 'test_group'], as_index=False).sum()
```
'total_group_trans' shows total revenue generateed by countrol and test group.
```
total_group_trans = (total_trans_df.drop('sample_id', axis =1)).groupby('test_group', as_index=False).sum()
```
```
crtl_avg_rev = (total_group_trans['transaction_amount'][0])/44886
test_avg_rev = (total_group_trans['transaction_amount'][1])/14835
```

On average, each user that was assigned to the **control group** generates about **2$** revenues.<br />
On the other hand, each user that was assigned to the **test group** generates **6.4$** average revenue.<br />
As a result a user that must **call-in** to cancel is **more likely** to generate more revenues.

## 4- Is a user that must call-in more likely to produce a higher chargeback rate(CHARGEBACKs/REBILLs)?
breakdown of transaction type numbers between the test group users:
```
test_trans_types = (df.drop(['transaction_id', 'transaction_amount'], axis =1)).groupby(['transaction_type', 'test_group'],).count().sort_values('test_group').iloc[3:]
```
![image](https://user-images.githubusercontent.com/65902301/178163401-f192d642-5170-40e7-b2b0-d1d151aa9829.png)

breakdown of transaction type numbers between the control group users:
```
ctrl_trans_types = (df.drop(['transaction_id', 'transaction_amount'], axis =1)).groupby(['transaction_type', 'test_group'],).count().sort_values('test_group').iloc[:3]
```
![image](https://user-images.githubusercontent.com/65902301/178163458-3290df52-ac30-4cc7-8654-117e39ac2c86.png)

Now the charback rate for each group is:
```
ctrl_chargback_rate = (ctrl_trans_types['sample_id'][0]/ctrl_trans_types['sample_id'][1])*100
test_chargeback_rate = (test_trans_types['sample_id'][0]/test_trans_types['sample_id'][1])*100
```
The chargeback rate in all the control group transactions is 2.82%.<br />
The chargeback rate in all the test group transactions is 1.78%.<br />
As a result A user that must call in is less likely to produce a higher chargeback rate.
