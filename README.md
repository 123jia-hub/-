# -工业故障预测数据处理分析及可视化
本项目由本人学习项目，有很多不足，往后会改善。
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import sqlalchemy
from pyecharts.charts import Bar3D
from pyecharts.commons.utils import JsCode
import pyecharts.options as  opts
from sqlalchemy import create_engine        
# 注意此行要填写本机信息engine = create_engine('mysql+pymysql://[用户名]:[密码]@[本机]:[端口]/[数据库]?charset=utf8')

#%%
df = pd.read_csv('D:\\工业故障预测系统的练习项目\\equipment_info.csv')

#%%
df.to_sql('equipment_info', engine, index=False)

#%%
df = pd.read_csv('D:\\工业故障预测系统的练习项目\\industrial_equipment_data.csv')
df
#%%
df.to_sql('industrial_equipment_data', engine, index=False)
#%%
df.info()
#%% md
# 计算设备运行的关键指标（如平均值、最大值、标准差等）
#%%
df1 = pd.read_sql("SELECT * FROM industrial_equipment_data", engine)
df1
#%%
df1

#%%

# 去重
df1 = df1.dropna()
# 特征取值
features = []
for ep_id in df1['设备ID'].unique():
    eq_data = df1[df1['设备ID'] == ep_id]
    # 字典形式记录数据
    equipment_data = {
        '设备ID':ep_id,
        '温度标准差':np.std(eq_data['温度(℃)']),
        '温度最大值':np.max(eq_data['温度(℃)']),
        '温度最小值':np.min(eq_data['温度(℃)']),
        '平均温度':np.mean(eq_data['温度(℃)']),
        '平均压力':np.mean(eq_data['压力(MPa)']),
        '压力标准差':np.std(eq_data['压力(MPa)']),
        '振动最大值':np.max(eq_data['振动(m/s²)']),
        '平均振动':np.mean(eq_data['振动(m/s²)']),
        '平均产量':np.mean(eq_data['产量(件)']),
        '平均合格率':np.mean(eq_data['合格率(%)']) 
    } 
    features.append(equipment_data)  
    
    # 转为Dataframe
features_df = pd.DataFrame(features)

features_df = features_df.round(2)
features_df
#%% md
## 数据可视化
#%%
from pyecharts.charts import Bar3D
from pyecharts import options as opts
import pandas as pd

# 假设您已经有了features_df DataFrame
# features_df = pd.DataFrame(features)  # 您已有的代码

# 准备3D柱状图数据
data = []
for index, row in features_df.iterrows():
    # 设备ID作为x轴
    device_id = row['设备ID']
    # 各种指标作为y轴
    metrics = [
        ('温度标准差', row['温度标准差']),
        ('温度最大值', row['温度最大值']),
        ('温度最小值', row['温度最小值']),
        ('平均温度', row['平均温度']),
        ('平均压力', row['平均压力']),
        ('压力标准差', row['压力标准差']),
        ('振动最大值', row['振动最大值']),
        ('平均振动', row['平均振动']),
        ('平均产量', row['平均产量']),
        ('平均合格率', row['平均合格率'])
    ]
    for metric_name, metric_value in metrics:
        data.append([device_id, metric_name, metric_value])

# 计算最大值用于视觉映射
range_max = features_df.iloc[:, 1:].max().max()

# 颜色范围
range_color = ['#313695', '#4575b4', '#74add1', '#abd9e9', '#e0f3f8', '#ffffbf', 
              '#fee090', '#fdae61', '#f46d43', '#d73027', '#a50026'] 

# 创建3D柱状图
c = ( 
    Bar3D()
    .add( 
        "",
        data,
        xaxis3d_opts=opts.Axis3DOpts(type_="category", name='设备ID'),
        yaxis3d_opts=opts.Axis3DOpts(type_="category", name='指标'),
        zaxis3d_opts=opts.Axis3DOpts(type_="value", name='数值'),
    ) 
    .set_global_opts(
        visualmap_opts=opts.VisualMapOpts(max_=range_max, range_color=range_color),
        title_opts=opts.TitleOpts(title="设备运行指标3D分析"),
    ) 
) 

# 渲染到本地文件
c.render("equipment_metrics_3d.html")

# 在notebook中显示
c.render_notebook()
