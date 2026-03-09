# SmartAd-AB-Testing-Analysis
评估不同投放策略对用户转化率的影响
## 一、实验背景
### 1.商业背景：
商家为了提高广告的点击率，设计了全新的智能广告，并以此开展A/B test，一组用户展示旧广告，一组展示新广告。
### 2.实验方式
通过A/B test的实验结果分析，判断新版广告是否会影响用户对于广告的点击率
原假设：Hₒ: p = pₒ - 两组广告的成功率没有显著差异
备择假设：Hₐ: p ≠ pₒ - 两组广告的成功率没有显著差异 ，采用双尾检验
置信水平：95% (α=0.05)
p 和 pₒ 分别代表新版广告与旧版广告的转化率

## 二、数据描述

## 三、数据描述
···import pandas as pd
import numpy as np
import scipy.stats as stats
from scipy.stats import norm
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.lines import Line2D
%matplotlib inline









