# SVC-日志分析
借助AI进行日志分析，脚本均来自AI生成，经过调试验证。
## 环境安装
### 基础环境配置
安装Python3，安装版本示例：
```
PS C:\Users\admin> python3
Python 3.13.9 (tags/v3.13.9:8183fa5, Oct 14 2025, 14:09:13) [MSC v.1944 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
```
安装pandas、openpyxl及matplotlib：
```shell
pip install pandas openpyxl matplotlib
```
### 验证环境
验证是否成功安装脚本：
```python
# 验证关键库是否安装成功
try:
	import pandas as pd
    import openpyxl
    import matplotlib.pyplot as plt
    print("=" * 40)
    print("✅ 所有库都安装成功啦！")
    print(f"pandas 版本: {pd.__version__}")
    print(f"openpyxl 版本: {openpyxl.__version__}")
    print(f"matplotlib 版本: {plt.matplotlib.__version__}")
    print("=" * 40)
except ImportError as e:
    print("❌ 有库没安装成功，错误信息:")
    print(e)
```
数据源是IBM TPC导出的性能数据，csv格式，名称示例：
```
PerfReport_000002032001530C_IOGroups_20251220-120000_24hrs0mins.csv
```
## 性能数据分析
###  IO Group CPU性能
I/O CPU使用趋势图代码：
```python
import pandas as pd
import matplotlib.pyplot as plt
import random

# -------------------------- 核心工具函数：Excel列标识转数字索引 --------------------------
def excel_col_to_index(col_str):
    """
    将Excel列标识（如A、C、Z、AA、AI）转换为从0开始的数字索引
    示例：A→0, C→2, G→6, Z→25, AA→26, AI→34
    """
    col_str = col_str.strip().upper()  # 转大写并去除空格
    index = 0
    for char in col_str:
        # 验证字符是否为字母
        if not char.isalpha():
            raise ValueError(f"列标识 '{col_str}' 包含非法字符，仅支持字母（A-Z）")
        # 计算：A=1, B=2...Z=26，最终减1转为0开始的索引
        index = index * 26 + (ord(char) - ord('A') + 1)
    return index - 1  # 转换为0开始的索引

# -------------------------- 1. 基础配置（新增分组列配置） --------------------------
# 文件路径
csv_file_path = "file/PerfReport_000002032001530C_IOGroups_20251220-120000_24hrs0mins.csv"
# 分组列配置（B列：io_grp0/io_grp1）
group_col_str = "B"                      # 分组信息所在列：第B列
group_column_name = "IO分组"              # 分组列显示名称
# 时间列配置（单列）
time_col_str = "C"                      # 采集时间所在列：第C列
time_column_name = "采集时间"            # 时间列显示名称
# 单指标配置（CPU利用率）
indicator_col_str_list = ["BB"]          # 指标列标识（BB列=系统CPU利用率）
indicator_column_name_list = [
    "系统 CPU 利用率"
]  # 指标显示名称

# 参考线配置（保留原有阈值线）
reference_lines = [
    {"value": 30, "color": "red", "linestyle": "--", "linewidth": 1, "label": "30%阈值线"},
    {"value": 60, "color": "orange", "linestyle": "--", "linewidth": 1, "label": "60%警告线"},
    {"value": 80, "color": "black", "linestyle": "--", "linewidth": 1, "label": "80%饱和线"}
]

# 分组配色+标记配置（区分io_grp0/io_grp1）
group_config = {
    "io_grp0": {
        "color": "#2E86AB",  # 蓝色
        "marker": "o",       # 圆形标记
        "label_suffix": "io_grp0"
    },
    "io_grp1": {
        "color": "#A23B72",  # 紫红色
        "marker": "s",       # 方形标记
        "label_suffix": "io_grp1"
    }
}

# 校验：列标识和列名数量必须一致
if len(indicator_col_str_list) != len(indicator_column_name_list):
    raise ValueError("指标列标识数量和列名数量不匹配！请检查配置")

# 转换列标识为数字索引（新增分组列转换）
try:
    group_col = excel_col_to_index(group_col_str)    # 分组列（B列）索引
    time_col = excel_col_to_index(time_col_str)      # 时间列（C列）索引
    indicator_col_index_list = [excel_col_to_index(col_str) for col_str in indicator_col_str_list]  # 指标列（BB列）索引
except ValueError as e:
    print(f"列标识转换失败：{e}")
    exit(1)

# -------------------------- 2. 读取并预处理数据（新增分组列处理） --------------------------
def load_and_preprocess_data(file_path):
    """读取CSV文件并进行数据预处理（含分组筛选、时间+指标清洗）"""
    try:
        # 构建需要读取的列索引：分组列 + 时间列 + 指标列
        usecols_list = [group_col] + [time_col] + indicator_col_index_list
        
        # 读取CSV文件（只读取必要列，提升效率）
        df = pd.read_csv(
            file_path,
            usecols=usecols_list,
            encoding='utf-8'  # 若文件编码为gbk，可改为encoding='gbk'
        )
        
        # 重命名列：分组列 + 时间列 + 指标列
        df.columns = [group_column_name] + [time_column_name] + indicator_column_name_list
        
        # 数据清洗步骤：
        # 1. 处理分组列：去空值 + 只保留配置中的有效分组（io_grp0/io_grp1）
        df = df.dropna(subset=[group_column_name])
        valid_groups = list(group_config.keys())
        df = df[df[group_column_name].isin(valid_groups)]  # 筛选有效分组数据
        if df.empty:
            print(f"警告：未找到有效分组数据（有效分组：{valid_groups}），请检查B列数据")
            return None
        
        # 2. 处理时间列：去空值 + 转换为datetime格式
        df = df.dropna(subset=[time_column_name])
        df[time_column_name] = pd.to_datetime(df[time_column_name], errors='coerce')
        df = df.dropna(subset=[time_column_name])  # 删除时间转换失败的行
        
        # 3. 处理指标列（CPU利用率）：转换为数值 + 去空值
        for indicator_col in indicator_column_name_list:
            df[indicator_col] = pd.to_numeric(df[indicator_col], errors='coerce')
            df = df.dropna(subset=[indicator_col])
        
        return df
    except Exception as e:
        print(f"读取/处理数据时出错：{e}")
        return None

# -------------------------- 3. 绘制曲线图（核心修改：按分组差异化绘图） --------------------------
def plot_performance_curve(df):
    """按IO分组绘制CPU利用率趋势图（io_grp0/io_grp1不同颜色+标记）"""
    # 设置中文字体（避免乱码）
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 中文黑体
    plt.rcParams['axes.unicode_minus'] = False    # 解决负号显示问题
    
    # 创建画布
    plt.figure(figsize=(14, 8))
    
    # 获取所有有效分组（去重）
    valid_groups = df[group_column_name].unique()
    
    # 绘制分组曲线：遍历每个分组+指标
    indicator_col = indicator_column_name_list[0]  # 当前仅CPU利用率一个指标
    for group in valid_groups:
        # 筛选当前分组的数据
        group_data = df[df[group_column_name] == group].copy()
        
        # 获取当前分组的配置（颜色、标记、标签）
        config = group_config[group]
        color = config["color"]
        marker = config["marker"]
        label = f"{indicator_col}-{config['label_suffix']}"  # 图例：系统CPU利用率-io_grp0
        
        # 绘制分组曲线
        plt.plot(
            group_data[time_column_name],  # x轴：采集时间
            group_data[indicator_col],     # y轴：CPU利用率
            color=color,                   # 分组专属颜色
            linewidth=1.8,                 # 线条宽度（略加粗，便于区分）
            marker=marker,                 # 分组专属标记
            markersize=5,                  # 标记大小
            label=label                    # 图例标签
        )
    
    # 绘制参考线（保留原有功能）
    for line in reference_lines:
        plt.axhline(
            y=line["value"],
            color=line["color"],
            linestyle=line["linestyle"],
            linewidth=line["linewidth"],
            label=line["label"],
            alpha=0.8  # 透明度，避免遮挡分组曲线
        )
    
    # 图表样式优化
    plt.title('SVC-I/O Group CPU性能（按IO分组）', fontsize=16, pad=20)
    plt.xlabel(time_column_name, fontsize=12)
    plt.ylabel(f'{indicator_col} (%)', fontsize=12)  # y轴标签添加百分号（CPU利用率专属）
    plt.xticks(rotation=45)  # 旋转x轴时间标签，避免重叠
    plt.grid(True, alpha=0.3, linestyle='--')  # 网格线（便于读数）
    plt.legend(loc='upper right', fontsize=10)  # 图例放在右上角（避免遮挡曲线）
    plt.tight_layout()  # 自动调整布局，防止标签截断
    
    # 保存+显示图片
    plt.savefig('SVC_IO分组_CPU性能趋势图.png', dpi=300, bbox_inches='tight')
    plt.show()

# -------------------------- 4. 主函数执行 --------------------------
if __name__ == "__main__":
    # 加载数据
    performance_data = load_and_preprocess_data(csv_file_path)
    
    if performance_data is not None and not performance_data.empty:
        # 打印数据基本信息（验证分组和指标）
        print("数据加载成功，前5行数据：")
        print(performance_data.head())
        print(f"\n数据总行数：{len(performance_data)}")
        print(f"时间范围：{performance_data[time_column_name].min()} 至 {performance_data[time_column_name].max()}")
        print(f"包含IO分组：{performance_data[group_column_name].unique()}")
        
        # 打印每个分组的CPU利用率范围
        indicator_col = indicator_column_name_list[0]
        print(f"\n{indicator_col}统计：")
        for group in performance_data[group_column_name].unique():
            group_data = performance_data[performance_data[group_column_name] == group]
            min_val = group_data[indicator_col].min()
            max_val = group_data[indicator_col].max()
            avg_val = group_data[indicator_col].mean()
            print(f"  {group}：最小值{min_val:.2f}% | 最大值{max_val:.2f}% | 平均值{avg_val:.2f}%")
        
        # 绘制曲线图
        plot_performance_curve(performance_data)
    else:
        print("数据加载失败或数据为空，请检查：")
        print("1. 文件路径是否正确（file/SC_SVC_iogrp.csv）")
        print("2. B列是否存在io_grp0/io_grp1分组数据（区分大小写）")
        print("3. BB列（系统CPU利用率）是否存在有效数值")
```
运行后结果示例（图片自动生成保存，此处不演示）：
```
PS C:\Users\admin\report-data> python ./iogrp-double.py
数据加载成功，前5行数据：
      IO分组                采集时间  系统 CPU 利用率
0  io_grp0 2025-12-20 12:00:00        4.85
1  io_grp1 2025-12-20 12:00:00       11.14
2  io_grp0 2025-12-20 12:05:00        4.41
3  io_grp1 2025-12-20 12:05:00       11.19
4  io_grp0 2025-12-20 12:10:00        4.08

数据总行数：576
时间范围：2025-12-20 12:00:00 至 2025-12-21 11:55:00
包含IO分组：['io_grp0' 'io_grp1']

系统 CPU 利用率统计：
  io_grp0：最小值3.70% | 最大值16.47% | 平均值6.41%
  io_grp1：最小值7.59% | 最大值31.41% | 平均值13.36%
```
### I/O Group读写响应时间
代码：
```python
import pandas as pd
import matplotlib.pyplot as plt
import random

# -------------------------- 核心工具函数：Excel列标识转数字索引 --------------------------
def excel_col_to_index(col_str):
    """
    将Excel列标识（如A、C、Z、AA、AI）转换为从0开始的数字索引
    示例：A→0, C→2, G→6, Z→25, AA→26, AI→34
    """
    col_str = col_str.strip().upper()  # 转大写并去除空格
    index = 0
    for char in col_str:
        # 验证字符是否为字母
        if not char.isalpha():
            raise ValueError(f"列标识 '{col_str}' 包含非法字符，仅支持字母（A-Z）")
        # 计算：A=1, B=2...Z=26，最终减1转为0开始的索引
        index = index * 26 + (ord(char) - ord('A') + 1)
    return index - 1  # 转换为0开始的索引

# -------------------------- 1. 基础配置（新增参考线配置） --------------------------
# 文件路径
csv_file_path = "file/PerfReport_0000020320615364_IOGroups_20251220-120000_24hrs0mins.csv"
# 时间列配置（单列）
time_col_str = "C"                      # 采集时间所在列：第C列
time_column_name = "采集时间"            # 时间列显示名称
# 多指标配置（数组/列表形式，一一对应）
indicator_col_str_list = ["O", "P", "Q"]  # 多个指标的Excel列标识（比如G=读缓存、H=写缓存、I=CPU使用率）
indicator_column_name_list = [
    "后端读响应时间 (ms/op)", 
    "后端写响应时间 (ms/op)", 
    "后端总响应时间 (ms/op)"
]  # 对应指标的显示名称

# 参考线配置（核心新增：支持多条参考线，可自定义数值、颜色、样式、标签）
reference_lines = [
    {"value": 5, "color": "red", "linestyle": "--", "linewidth": 1, "label": "5ms关注线"},
    # 可添加更多参考线，示例：
    {"value": 10, "color": "orange", "linestyle": "--", "linewidth": 1, "label": "10ms警戒线"},
]

# 校验：列标识和列名数量必须一致
if len(indicator_col_str_list) != len(indicator_column_name_list):
    raise ValueError("指标列标识数量和列名数量不匹配！请检查配置")

# 转换指标列标识为数字索引（列表推导式）
try:
    time_col = excel_col_to_index(time_col_str)
    indicator_col_index_list = [excel_col_to_index(col_str) for col_str in indicator_col_str_list]
except ValueError as e:
    print(f"列标识转换失败：{e}")
    exit(1)

# -------------------------- 2. 读取并预处理数据（逻辑不变） --------------------------
def load_and_preprocess_data(file_path):
    """读取CSV文件并进行数据预处理（支持多指标）"""
    try:
        # 构建需要读取的列索引：时间列 + 所有指标列
        usecols_list = [time_col] + indicator_col_index_list
        
        # 读取CSV文件（只读取需要的列，提升效率）
        df = pd.read_csv(
            file_path,
            usecols=usecols_list,  # 多列索引
            encoding='utf-8'  # 根据文件实际编码调整，如gbk
        )
        
        # 重命名列：时间列 + 所有指标列
        df.columns = [time_column_name] + indicator_column_name_list
        
        # 数据清洗：先处理时间列
        df = df.dropna(subset=[time_column_name])
        df[time_column_name] = pd.to_datetime(df[time_column_name], errors='coerce')
        df = df.dropna(subset=[time_column_name])
        
        # 循环处理每个指标列（数值转换+空值删除）
        for indicator_col in indicator_column_name_list:
            df[indicator_col] = pd.to_numeric(df[indicator_col], errors='coerce')
            df = df.dropna(subset=[indicator_col])
        
        return df
    except Exception as e:
        print(f"读取/处理数据时出错：{e}")
        return None

# -------------------------- 3. 绘制曲线图（逻辑不变，无标注） --------------------------
def plot_performance_curve(df):
    """绘制多指标随时间变化的曲线图（含自定义参考线）"""
    # 设置中文字体（避免乱码）
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 中文黑体
    plt.rcParams['axes.unicode_minus'] = False    # 解决负号显示问题
    
    # 创建画布
    plt.figure(figsize=(14, 8))
    
    # 定义配色方案（避免颜色重复，可扩展）
    color_list = ['#2E86AB', '#A23B72', '#F18F01', '#C73E1D', '#7209B7', '#0B4F6C']
    # 标记样式列表
    marker_list = ['.', 'o', 's', '^', 'x', '+']
    
    # 循环绘制每个指标的曲线
    for idx, indicator_col in enumerate(indicator_column_name_list):
        # 选择颜色和标记（超出预设则随机色）
        color = color_list[idx] if idx < len(color_list) else (random.random(), random.random(), random.random())
        marker = marker_list[idx] if idx < len(marker_list) else '.'
        
        # 绘制单条曲线
        plt.plot(
            df[time_column_name],        # x轴：采集时间
            df[indicator_col],           # y轴：当前指标
            color=color,                 # 曲线颜色
            linewidth=1.5,               # 线条宽度
            marker=marker,               # 数据点标记
            markersize=4,                # 标记大小
            label=indicator_col          # 图例标签（区分不同指标）
        )
    
    # -------------------------- 绘制参考线 --------------------------
    for line in reference_lines:
        plt.axhline(
            y=line["value"],            # 参考线数值（y轴）
            color=line["color"],        # 参考线颜色
            linestyle=line["linestyle"],# 参考线样式（--虚线、:点线、-实线）
            linewidth=line["linewidth"],# 参考线宽度
            label=line["label"],        # 参考线图例标签
            alpha=0.8                   # 透明度（避免遮挡指标曲线）
        )
    
    # 设置图表样式
    plt.title('SVC-IO-Group读写响应时间曲线', fontsize=16, pad=20)
    plt.xlabel(time_column_name, fontsize=12)
    plt.ylabel('指标数值', fontsize=12)  # 统一y轴标签（多指标数值单位可能不同）
    
    # 旋转x轴时间标签，避免重叠
    plt.xticks(rotation=45)
    
    # 添加网格（便于读数）
    plt.grid(True, alpha=0.3, linestyle='--')
    
    # 添加图例（包含指标曲线+参考线）
    plt.legend(loc='best', fontsize=10)
    
    # 自动调整布局（防止标签/图例被截断）
    plt.tight_layout()
    
    # 保存图片（可选，也可以用plt.show()直接显示）
    plt.savefig('多性能指标趋势图.png', dpi=300, bbox_inches='tight')
    # 显示图表
    plt.show()

# -------------------------- 4. 主函数执行（仅控制台输出统计信息） --------------------------
if __name__ == "__main__":
    # 加载数据
    performance_data = load_and_preprocess_data(csv_file_path)
    
    if performance_data is not None and not performance_data.empty:
        # 打印数据基本信息（便于验证）
        print("数据加载成功，前5行数据：")
        print(performance_data.head())
        print(f"\n数据总行数：{len(performance_data)}")
        print(f"时间范围：{performance_data[time_column_name].min()} 至 {performance_data[time_column_name].max()}")
        
        # 循环打印每个指标的 最小值、最大值、平均值
        for indicator_col in indicator_column_name_list:
            min_val = performance_data[indicator_col].min()
            max_val = performance_data[indicator_col].max()
            avg_val = performance_data[indicator_col].mean()
            print(f"\n{indicator_col}统计信息：")
            print(f"  最小值：{min_val:.2f}")
            print(f"  最大值：{max_val:.2f}")
            print(f"  平均值：{avg_val:.2f}")
        
        # 绘制曲线图
        plot_performance_curve(performance_data)
    else:
        print("数据加载失败或数据为空，请检查文件路径和列标识是否正确！")
```
输出示例：
```
PS C:\Users\admin\report-data> python .\iogrp-time.py
数据加载成功，前5行数据：
                 采集时间  后端读响应时间 (ms/op)  后端写响应时间 (ms/op)  后端总响应时间 (ms/op)
0 2025-12-20 12:03:00            0.925            0.342            0.604
1 2025-12-20 12:08:00            1.280            0.396            0.856
2 2025-12-20 12:13:00            0.957            0.379            0.638
3 2025-12-20 12:18:00            1.163            0.303            0.864
4 2025-12-20 12:23:00            0.796            0.380            0.538

数据总行数：286
时间范围：2025-12-20 12:03:00 至 2025-12-21 11:57:00

后端读响应时间 (ms/op)统计信息：
  最小值：0.62
  最大值：13.23
  平均值：1.74

后端写响应时间 (ms/op)统计信息：
  最小值：0.30
  最大值：14.11
  平均值：0.65

后端总响应时间 (ms/op)统计信息：
  最小值：0.48
  最大值：11.36
  平均值：1.49
```
### 数据对比
```python
import pandas as pd
import os

def analyze_two_groups_excel():
    """
    读取当前目录/file文件夹下的Excel，结果输出到当前目录
    输入路径：./file/iogrep host.xlsx（当前目录下的file子文件夹）
    输出路径：./修正版_两组数据对比分析结果.xlsx（当前工作目录）
    """
    # 1. 定义固定路径（按需求设置）
    # 输入路径：当前目录下的file文件夹 + 目标Excel文件名
    input_dir = "./file"  # 当前目录下的file子文件夹
    input_filename = "iogrep host.xlsx"  # 目标Excel文件名
    input_file = os.path.join(input_dir, input_filename)
    
    # 输出路径：当前目录 + 结果文件名
    output_file = "./修正版_两组数据对比分析结果.xlsx"  # 当前目录输出
    
    # 2. 验证输入路径是否存在
    if not os.path.exists(input_dir):
        raise FileNotFoundError(f"输入文件夹不存在：{input_dir}\n请在当前目录下创建'file'文件夹，并放入Excel文件")
    
    if not os.path.exists(input_file):
        raise FileNotFoundError(f"Excel文件不存在：{input_file}\n请确认file文件夹内是否有'{input_filename}'文件")
    
    # 3. 读取Excel文件并预处理
    df_raw = pd.read_excel(input_file)
    df_raw = df_raw.rename(columns={'id': 'A', 'name': 'B', 'id.1': 'C', 'name.1': 'D'})
    
    # 4. 数据清洗：筛选有效数据（排除空值）
    # 处理A、B列（第一组）
    ab_group = df_raw[['A', 'B']].copy()
    ab_group = ab_group.dropna(subset=['A', 'B'])  # 仅保留A、B都不为空的行
    ab_group['A'] = ab_group['A'].astype(int).astype(str)  # 统一ID为字符串格式（避免数字类型差异）
    ab_group['AB_key'] = ab_group['A'] + '|' + ab_group['B'].astype(str)  # 组合键用于匹配
    
    # 处理C、D列（第二组）
    cd_group = df_raw[['C', 'D']].copy()
    cd_group = cd_group.dropna(subset=['C', 'D'])  # 仅保留C、D都不为空的行
    cd_group['C'] = cd_group['C'].astype(str)  # 统一ID格式
    cd_group['CD_key'] = cd_group['C'] + '|' + cd_group['D'].astype(str)  # 组合键用于匹配
    
    # 5. 集合匹配计算（核心逻辑）
    ab_keys = set(ab_group['AB_key'].unique())  # A、B组所有唯一组合
    cd_keys = set(cd_group['CD_key'].unique())  # C、D组所有唯一组合
    
    # 计算交集（两组相同）、差集（各组独有）
    common_keys = ab_keys.intersection(cd_keys)  # 相同数据
    ab_only_keys = ab_keys.difference(cd_keys)   # AB组独有
    cd_only_keys = cd_keys.difference(ab_keys)   # CD组独有
    
    # 6. 构建结果数据
    # 6.1 两组相同数据（A/B、C/D列都填充）
    common_result = []
    for key in common_keys:
        ab_row = ab_group[ab_group['AB_key'] == key].iloc[0]
        cd_row = cd_group[cd_group['CD_key'] == key].iloc[0]
        common_result.append({
            'A': ab_row['A'], 'B': ab_row['B'],
            'C': cd_row['C'], 'D': cd_row['D'],
            '匹配状态': '两组相同'
        })
    
    # 6.2 AB组独有数据（C/D列为空）
    ab_only_result = []
    for key in ab_only_keys:
        ab_row = ab_group[ab_group['AB_key'] == key].iloc[0]
        ab_only_result.append({
            'A': ab_row['A'], 'B': ab_row['B'],
            'C': '', 'D': '',
            '匹配状态': 'AB组独有'
        })
    
    # 6.3 CD组独有数据（A/B列为空）
    cd_only_result = []
    for key in cd_only_keys:
        cd_row = cd_group[cd_group['CD_key'] == key].iloc[0]
        cd_only_result.append({
            'A': '', 'B': '',
            'C': cd_row['C'], 'D': cd_row['D'],
            '匹配状态': 'CD组独有'
        })
    
    # 7. 合并结果并生成统计
    result_df = pd.DataFrame(common_result + ab_only_result + cd_only_result)
    
    stats = {
        'AB组有效行数': len(ab_group),
        'CD组有效行数': len(cd_group),
        '两组相同行数': len(common_result),
        'AB组独有行数': len(ab_only_result),
        'CD组独有行数': len(cd_only_result)
    }
    stats_df = pd.DataFrame(list(stats.items()), columns=['统计项目', '数量'])
    
    # 8. 保存结果到当前目录
    with pd.ExcelWriter(output_file, engine='openpyxl') as writer:
        result_df.to_excel(writer, sheet_name='数据对比结果', index=False)
        stats_df.to_excel(writer, sheet_name='统计信息', index=False)
        ab_group.to_excel(writer, sheet_name='AB组原始数据', index=False)
        cd_group.to_excel(writer, sheet_name='CD组原始数据', index=False)
    
    # 9. 打印执行结果
    print("="*50)
    print("✅ 数据对比分析完成！")
    print(f"📥 输入文件：{input_file}")
    print(f"📤 输出文件：{os.path.abspath(output_file)}")  # 显示绝对路径便于查找
    print("\n📊 统计结果：")
    for item, count in stats.items():
        print(f"   • {item}：{count}")
    print("="*50)
    
    return result_df, stats

# 直接执行分析（无需额外调用）
if __name__ == "__main__":
    try:
        analyze_two_groups_excel()
    except Exception as e:
        print(f"❌ 执行出错：{str(e)}")
```
输出示例：
```
PS C:\Users\admin\report-data> python .\iogrp-host.py   
==================================================
✅ 数据对比分析完成！
📥 输入文件：./file\iogrep host.xlsx
📤 输出文件：C:\Users\admin\report-data\修正版_两组数据对比分析结果.xlsx

📊 统计结果：
   • AB组有效行数：168
   • CD组有效行数：184
   • 两组相同行数：167
   • AB组独有行数：1
   • CD组独有行数：17
==================================================
```
### 去重找出没有镜像的vdisk
脚本：
```python
import pandas as pd
import os

def remove_duplicate_rows_by_column_position():
    """
    按Excel列位置（编号）去重（通用版）：
    - 不依赖列名，按列位置定位：A=第1列(索引0)、B=第2列(索引1)、C=第3列(索引2)...
    - 以B列（索引1）为基准，删除所有包含重复值的行
    - 仅保留B列值唯一的行，保留原始所有列结构
    """
    # 1. 配置参数（可根据需求修改）
    input_file = "./file/vdiskcopy1.xlsx"  # 输入文件路径
    output_file = "./B列位置去重结果.xlsx"  # 输出文件路径
    target_column_index = 1                # 目标列位置索引（B列=1，A列=0，C列=2，以此类推）
    
    # 2. 验证输入文件是否存在
    if not os.path.exists(input_file):
        print(f"❌ 错误：找不到输入文件！")
        print(f"文件路径：{input_file}")
        return
    
    # 3. 读取Excel文件（不指定列名，按原始列位置处理）
    try:
        df = pd.read_excel(input_file, header=0)  # header=0表示第一行为列名，不影响列位置定位
        print(f"✅ 成功读取文件：{input_file}")
        print(f"📊 原始数据：{len(df)} 行，{len(df.columns)} 列")
        print(f"🔍 列位置与列名对应关系：")
        for idx, col_name in enumerate(df.columns):
            excel_col = chr(65 + idx)  # 转换为Excel列名：0→A，1→B，2→C...
            print(f"   列{excel_col}（索引{idx}）：{col_name}")
    except Exception as e:
        print(f"❌ 读取文件失败：{str(e)}")
        return
    
    # 4. 验证目标列位置是否有效
    if target_column_index < 0 or target_column_index >= len(df.columns):
        print(f"\n❌ 错误：目标列位置无效！")
        print(f"当前文件共有 {len(df.columns)} 列（A列到 {chr(65 + len(df.columns)-1)} 列）")
        print(f"请输入0-{len(df.columns)-1}之间的索引（B列=1）")
        return
    
    # 5. 获取目标列信息（B列）
    target_column_name = df.columns[target_column_index]
    excel_column_name = chr(65 + target_column_index)  # 转换为Excel列名（如1→B）
    print(f"\n🎯 目标去重列：Excel列{excel_column_name}（索引{target_column_index}），列名：{target_column_name}")
    
    # 6. 以B列为基准去重（核心逻辑）
    # 统计目标列各值的出现次数
    value_counts = df[target_column_name].value_counts()
    
    # 筛选出出现次数 == 1 的唯一值
    unique_values = value_counts[value_counts == 1].index.tolist()
    
    # 保留目标列值为唯一值的所有行
    filtered_df = df[df[target_column_name].isin(unique_values)].copy()
    
    # 7. 统计去重结果
    removed_count = len(df) - len(filtered_df)
    duplicate_values = value_counts[value_counts > 1].index.tolist()[:10]  # 只显示前10个重复值
    
    # 8. 保存筛选结果
    try:
        filtered_df.to_excel(output_file, index=False)
        print(f"\n✅ 去重完成！结果已保存至：{os.path.abspath(output_file)}")
    except Exception as e:
        print(f"❌ 保存文件失败：{str(e)}")
        return
    
    # 9. 打印详细统计信息
    print("\n" + "="*80)
    print(f"📈 去重统计结果（基准列：Excel列{excel_column_name}）")
    print(f"   • 原始总行数：{len(df)}")
    print(f"   • 去重后总行数：{len(filtered_df)}")
    print(f"   • 被删除的重复行数：{removed_count}")
    print(f"   • 重复值数量：{len(value_counts[value_counts > 1])} 个")
    print(f"   • 前10个重复值示例：{duplicate_values}")
    if len(duplicate_values) >= 10:
        print(f"   • （更多重复值已省略，可查看完整统计）")
    print("="*80)

# 直接执行脚本
if __name__ == "__main__":
    remove_duplicate_rows_by_column_position()
```
运行示例：
```
PS C:\Users\admin\report-data> python .\vdiskcopy.py
✅ 成功读取文件：./file/vdiskcopy1.xlsx
📊 原始数据：2197 行，17 列
🔍 列位置与列名对应关系：
   列A（索引0）：vdisk_id
   列B（索引1）：vdisk_name
   列C（索引2）：copy_id
   列D（索引3）：status
   列E（索引4）：sync
   列F（索引5）：primary
   列G（索引6）：mdisk_grp_id
   列H（索引7）：mdisk_grp_name
   列I（索引8）：capacity
   列J（索引9）：type
   列K（索引10）：se_copy
   列L（索引11）：easy_tier
   列M（索引12）：easy_tier_status
   列N（索引13）：compressed_copy
   列O（索引14）：parent_mdisk_grp_id
   列P（索引15）：parent_mdisk_grp_name
   列Q（索引16）：encrypt

🎯 目标去重列：Excel列B（索引1），列名：vdisk_name

✅ 去重完成！结果已保存至：C:\Users\admin\report-data\B列位置去重结果.xlsx

================================================================================
📈 去重统计结果（基准列：Excel列B）
   • 原始总行数：2197
   • 去重后总行数：445
   • 被删除的重复行数：1752
   • 重复值数量：876 个
   • 前10个重复值示例：['ZZD000017YXPT_vol51_1T', 'ac17aepdb_data_vol9', 'ZZD170000XDFXGL_100G_vol09', 'ac17aepdb_data_vol8', 'dc17aepzxdb1_data_vol0', 'ac17aepdb_data_vol7', 'dc17cbrmsdb_ocr_vol0', 'dc17farmsdb_arch_vol0', 'dc17farmsdb_data_vol3', 'dc17farmsdb_data_vol2']
   • （更多重复值已省略，可查看完整统计）
================================================================================
```
## 待补充