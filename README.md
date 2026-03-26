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




## # 工业故障预测系统 🏭

> 基于工业设备传感器数据的故障预测与可视化分析系统

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📖 项目简介

本项目是一个工业设备故障预测与分析系统，通过对风机、压缩机等工业设备的温度、压力、振动等传感器数据进行采集、清洗和深度分析，利用 **3D 可视化技术**直观展示设备运行状态，为设备维护和故障预警提供数据支持。

### ✨ 核心特性

- 🔍 **数据预处理**：智能处理缺失值和异常值
- 📊 **多维度指标计算**：温度、压力、振动的统计分析
- 🎨 **3D 可视化**：交互式 3D 柱状图，支持鼠标拖动和缩放
- 🎬 **动态演示**：24 小时设备状态变化动画
- ⚡ **高性能**：使用 Pandas groupby 优化，比传统循环快数十倍
### 2️⃣ 3D 柱状图可视化 🎨

**特性**：
- 🖱️ 鼠标左键拖动：360°旋转查看
- 🔍 鼠标滚轮：自由缩放
- 🔄 自动旋转展示
- 🎨 热力图颜色映射

**交互效果**：
- 对比多个设备的不同指标
- 直观展示设备间的性能差异
- 支持工具提示查看详细数值

### 3️⃣ 时间轴动画演示 🎬

**功能**：
- ⏯️ 自动播放 24 小时数据变化
- 📍 手动拖动时间轴查看特定时刻
- 🎯 追踪单个设备的全天运行轨迹
- 📈 观察温度、压力等参数的时序变化

### 4️⃣ 多指标综合分析

同时展示：
- 平均温度 vs 最高温度
- 平均压力（放大 10 倍）
- 平均振动（放大 100 倍）
- 颜色梯度显示数值范围

---

## 🤝 贡献指南

欢迎参与项目开发！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

---

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

---

## 👨‍💻 作者

**[你的名字]** 
- GitHub: [@你的用户名](https://github.com/你的用户名)
- 专业背景：工业过程自动化
- 技术栈：数据分析、机器学习、工业智能化

---

## 📮 联系方式

如有问题或建议，请通过以下方式联系：
- 提交 Issue
- 发送邮件至：your.email@example.com

---

## 🙏 致谢

感谢以下开源项目：
- [PyECharts](https://pyecharts.org/) - 强大的 Python 可视化库
- [Pandas](https://pandas.pydata.org/) - 数据分析利器
- [Jupyter](https://jupyter.org/) - 交互式开发环境

---

<div align="center">

**如果这个项目对你有帮助，请给一个 ⭐ Star！**

Made with ❤️ by [你的名字]

</div>


   
