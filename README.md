# recruitment-python-DI-category-articles-homework-ZeqiaoChen# 导入客户数据
## 导入pandas库
#import pandas as pd
## 读取test.csv
#test_csv = 
## 查看test_csv
#test_csv
import warnings
# 关闭警告显示
warnings.filterwarnings('ignore')


# 导入客户数据
customer_info = pd.read('./mnt/log/example.com/access_20210719.csv.gz')
# 查看数据的基本信息总结
data.info()

# 清洗【uri】列的缺失值
data = data.dropna(subset=['uri'])
# 查看清洗后的数据基本信息总结
data.info()

# 查找重复数据
data[data.duplicated()]
# 删除重复值
data = data.drop_duplicates()
# 查找清洗后的数据是否存在重复数据
data[data.duplicated()]
# 查看数据描述性统计的信息
data.describe()
# 筛选【level】列大于 0 的数据
data = data[(data['level'] > 0)]
# 查看数据描述性统计的信息
data.describe()




# 按【uid】和【host】分组后，获取【datetime】列的最大值和【level】列的总和
grouped_data = data.groupby(['uid', 'host'], as_index=False).agg({'datetime': 'max', 'level': 'sum'})
grouped_data.head(10)

# 计算时间间隔（天数）
today = '2021-11-24 00:00:00'
pd.to_datetime(today) - pd.to_datetime(grouped_data['datetime'])

# 计算时间间隔
today = '2021-11-24 00:00:00'
grouped_data['时间间隔'] = (pd.to_datetime(today) - pd.to_datetime(grouped_data['datetime'])).dt.days
grouped_data


# 按【host】分组后，获取【时间间隔】列的最小值、【uid】列的数量，以及【level】列的总和
rfm_data = grouped_data.groupby('host', as_index=False).agg({'时间间隔': 'min', 'uid': 'count', 'level': 'sum'})



# 修改列名为：host、时间间隔、uid和 sum of level
rfm_data.columns = ['host', '时间间隔', 'uid', 'level']

# 设置中文显示字体
plt.rcParams['font.family'] = ['Source Han Sans CN']

# 画 R 值（时间间隔）的折线图，便于后续对数据进行分组
plt.figure(figsize=(10, 8))
x = rfm_data['时间间隔'].sort_values()
y = rfm_data.index
plt.plot(x, y)
rfm_data

# 按【host】分组后，获取【时间间隔】列的最小值、【uid】列的数量，以及【level】列的总和
rfm_data = grouped_data.groupby('用户 ID', as_index=False).agg({'时间间隔': 'min', '订单号': 'count', '总金额': 'sum'})

# 修改列名为：host、时间间隔、Total Number of uid和level
rfm_data.columns = ['host', '时间间隔', 'Total Number of uid', 'level']
rfm_data

# 设置中文显示字体
plt.rcParams['font.family'] = ['Source Han Sans CN']

# 画 R 值（时间间隔）的折线图，便于后续对数据进行分组
plt.figure(figsize=(10, 8))
x = rfm_data['时间间隔'].sort_values()
y = rfm_data.index
plt.plot(x, y)

# 定义函数按照区间划分 R 值
def caculate_r(s):
    if s <= 100:
        return 5
    elif s <= 200:
        return 4
    elif s <= 300:
        return 3
    elif s <= 400:
        return 2
    else:
        return 1

# 对 R 值进行评分
rfm_data['R评分'] = rfm_data['时间间隔'].agg(caculate_r)
rfm_data

# 画 F 值（Total Number of uid）的折线图，便于后续对数据进行分组
plt.figure(figsize=(10, 8))
x = rfm_data['Total Number of uid'].sort_values()
y = rfm_data.index
plt.plot(x, y)

# 定义函数按照区间划分 F 值
def caculate_f(s):
    if s <= 5:
        return 1
    elif s <= 10:
        return 2
    elif s <= 15:
        return 3
    elif s <= 20:
        return 4
    else:
        return 5

# 对 F 值进行评分
rfm_data['F评分'] = rfm_data['Total Number of uid'].agg(caculate_f)
rfm_data

# 画 M 值（level）的折线图，便于后续对数据进行分组
plt.figure(figsize=(10, 8))
x = rfm_data['level'].sort_values()
y = rfm_data.index
plt.plot(x, y)

# 定义函数按照区间划分 M 值
def caculate_m(s):
    if s <= 2000:
        return 1
    elif s <= 4000:
        return 2
    elif s <= 6000:
        return 3
    elif s <= 8000:
        return 4
    else:
        return 5

# 对 M 值进行评分
rfm_data['M评分'] = rfm_data['level'].agg(caculate_m)
rfm_data

# 计算 R评分、F评分、M评分的平均数
r_avg = rfm_data['R评分'].mean()
f_avg = rfm_data['F评分'].mean()
m_avg = rfm_data['M评分'].mean()

print('R评分的均值为：{}，F评分的均值为{}，M评分的均值为{}'.format(r_avg, f_avg, m_avg))

# 查看 R 值的分数与阈值对比后得到的返回值
rfm_data['R评分'] > r_avg

num1 = bool(1) * 1
num2 = bool(0) * 1
print('num1 的值为{}，num2 的值为{}'.format(num1, num2))

# 将R评分、F评分、M评分 的数据分别与对应的平均数做比较
rfm_data['R评分'] = (rfm_data['R评分'] > r_avg) * 1
rfm_data['F评分'] = (rfm_data['F评分'] > f_avg) * 1
rfm_data['M评分'] = (rfm_data['M评分'] > m_avg) * 1
rfm_data

# 拼接R评分、F评分、M评分
rfm_score = rfm_data['R评分'].astype(str) + rfm_data['F评分'].astype(str) + rfm_data['M评分'].astype(str)
rfm_score

# 定义字典标记 RFM 评分档对应的用户分类名称
transform_label = {
    '111':'重要整治网页',
    '101':'中度整治网页',
    '011':'轻度整治网页',
    '001':'微整治网页',
    '110':'重要关注网页',
    '100':'中度关注网页',
    '010':'优秀网页',
    '000':'超一流网页'
}
