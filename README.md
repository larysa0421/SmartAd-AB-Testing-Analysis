# SmartAd-AB-Testing-Analysis
评估不同投放策略对用户转化率的影响
## 一、实验背景
### 1.商业背景：
线上的用户被提供BIO问卷，商家为了提高用户的参与度，设计了全新的智能广告，并以此开展A/B test，一组用户展示旧广告control（对照组），一组展示新广告exposed（实验组）。
### 2.实验方式
#### 通过A/B test的实验结果分析，判断新版广告是否会影响用户填写问卷的转化率
#### 原假设：Hₒ: p = pₒ - 两组广告的成功率没有显著差异
#### 备择假设：Hₐ: p ≠ pₒ - 两组广告的成功率没有显著差异 ，采用双尾检验
#### 置信水平：95% (α=0.05)
#### p 和 pₒ 分别代表新版广告与旧版广告的转化率

## 二、数据描述
<img width="2012" height="368" alt="image" src="https://github.com/user-attachments/assets/0f9ae4d7-5daf-42da-9784-9d952b01cbed" />

``` import pandas as pd
import numpy as np
import scipy.stats as stats
from scipy.stats import norm
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.lines import Line2D
%matplotlib inline
df = pd.read_csv('./AdSmartABdata - AdSmartABdata.csv')
df.head()
df.info()
```

## 三、数据预处理
### 1.字段定义：
明确 auction_id 作为用户唯一标识，experiment 作为分组标签
### 2.对数据的空值、重复值进行预处理
### 3.响应转换：
通过自定义函数 get_response 将 yes 和 no 列整合，识别出真正的响应用户，剔除了未作答样本（no response）

``` df.isnull().sum()
df.shape
df.columns
num_duplicates = sum(df.duplicated(subset=['auction_id']))
print(f'Amount of duplicates: {num_duplicates}')
def get_response(row):
    if row[0] == 1:
        res = 'yes'
    elif row[1] == 1:
        res = 'no'
    else:
        res = 'no response'
        
    return res
```
## 四、实验组和对照组的表现比较
### 为了从不同维度评估实验效果，本项目对比了两种统计口径
### 1.全量参与：按照对照组和实验组，把响应情况可视化，并计算计算不同响应情况的占比
### 2.净转化：剔除 no response后，把响应情况可视化，并计算计算不同响应情况的占比

```
## 全量参与可视化
df['response'] = df[['yes', 'no']].apply(get_response, axis=1)
sns.displot(df.sort_values(by='experiment'), x='response', 
            col='experiment', hue='experiment', 
            palette=['#1CD631','#A757EE'], alpha=1)
plt.show()
df_response = pd.pivot_table(data=df, index='experiment', columns='response', aggfunc='count')['auction_id']
df_response['total'] = df_response.apply(sum, axis=1)
df_response = df_response.apply(get_category_percent, axis=1)
df_response.columns.name = 'response %'
df_response = df_response[['yes', 'no', 'no response']]
## 全量参与占比
display(df_response)

def get_category_percent(row, decimal=1):
    return round(row/max(row)*100, decimal)

## 净转化可视化
df = df[(df['yes'] == 1) | (df['no'] == 1)]
sns.displot(df.sort_values(by='experiment'), x='response', 
            col='experiment', hue='experiment', 
            palette=['#1CD631','#A757EE'], alpha=1, height=3, aspect=0.9)

plt.show()
df_response = pd.pivot_table(data=df, index='experiment', columns='response', aggfunc='count')['auction_id']
df_response['total'] = df_response.apply(sum, axis=1)
df_response = df_response.apply(get_category_percent, axis=1)
df_response.columns.name = 'response %'
df_response = df_response[['yes', 'no']]
display(df_response)
```

## 五、统计显著性检验
### 1.统计分流，确认分流方案是否均匀，明确后续的检验结果是否可信
输出结果为47.1%和52.9%，在45%-55%的区间内，说明分组是随机且合理的
### 2.确定实验周期
整个 A/B 测试从 2020 年7月3日持续到了2020 年7月10日，共8天
### 3.观察每日样本的变化量
观察两组曲线是否贴合，保证流量分组的均匀和

```
#统计分流
sns.catplot(data=df.sort_values(by='experiment'), x='experiment', 
            kind='count', height=3, palette=['#1CD631','#A757EE'])\
            .set(title='Amount of users in exposed and control groups')
plt.show()
print('Percent of users in control group: {:.1%}'\
      .format(len(df[df['experiment']=='control'])/len(df)))
print('Percent of users in exposed group: {:.1%}'\
      .format(len(df[df['experiment']=='exposed'])/len(df)))
#确认实验周期
print('First date of experiment: {}'.format(df['date'].min()))
print('Last date of experiment:  {}'.format(df['date'].max()))
#观察每日样本变化量
fig, ax = plt.subplots(figsize=(10,5))
g = sns.lineplot(data=df.groupby(['experiment', 'date'])['date'].count()['control'], 
                ax=ax, label='control', color='#1CD631', linewidth = 4)
g = sns.lineplot(data=df.groupby(['experiment', 'date'])['date'].count()['exposed'], 
                ax=ax, label='exposed', color='#A757EE', linewidth = 4)
ax.grid(True)
ax.set_ylabel('count')
ax.set_title('Amount of users versus date')
plt.show()
```








