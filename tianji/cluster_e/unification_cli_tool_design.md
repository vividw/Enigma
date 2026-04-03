# 命令行玄学工具设计模式统一分析

**分析日期**: 2025年  
**分析范围**: Node.js/Go/Rust/Python CLI工具  
**分析版本**: v1.0  

---

## 一、分析背景

### 1.1 分析目的

本分析旨在总结命令行玄学工具的设计模式，为开发者提供统一的CLI设计参考。分析涵盖qimen-cli、bazi-go、lunar-rust、fengshui-python等命令行工具的设计实践。

### 1.2 分析对象

**qimen-cli** (Node.js)

- 奇门遁甲命令行工具
- Commander.js框架

**bazi-go** (Go)

- 八字计算库
- 标准库flag

**lunar-rust** (Rust)

- 农历计算库
- clap框架

**fengshui-python** (Python)

- 风水计算工具集
- Click框架

---

## 二、设计模式对比

### 2.1 命令结构模式

**子命令模式**

所有工具均采用子命令模式组织功能:

```
# qimen-cli (Node.js)
qimen calc --date 2024-01-01
qimen show --id 123
qimen export --format json

# bazi-go (Go)
bazi calc --date 2024-01-01
bazi dayun --count 10
bazi analyze --type wuxing

# lunar-rust (Rust)
lunar convert --from solar --to lunar 2024-01-01
lunar term --year 2024
lunar ganzhi --date 2024-01-01

# fengshui-python (Python)
fengshui compass --heading 180
fengshui xuankong --period 8 --mountain zi
fengshui bazhai --minggua 1
```

**设计评价**

- **优点**: 功能清晰，易于扩展
- **缺点**: 命令层级较深时输入繁琐

### 2.2 参数传递模式

**POSIX风格参数**

```bash
# 短参数
-q query -d 2024-01-01

# 长参数
--query "求财" --date "2024-01-01"

# 混合使用
-q "求财" --date "2024-01-01"
```

**JSON输入模式**

```bash
# 复杂参数通过JSON传递
cat input.json | qimen calc --stdin

# 或
qimen calc --json '{"date":"2024-01-01","query":"求财"}'
```

**交互式输入**

```bash
# 交互式提示
qimen calc --interactive
# ? 请输入日期: 2024-01-01
# ? 请输入问题: 求财
```

### 2.3 输出格式模式

**多格式支持**

```bash
# 表格格式 (默认)
qimen calc --format table

# JSON格式
qimen calc --format json

# CSV格式
qimen calc --format csv

# Markdown格式
qimen calc --format markdown
```

**颜色输出**

```bash
# 启用颜色
qimen calc --color

# 禁用颜色
qimen calc --no-color

# 自动检测
qimen calc --color auto
```

### 2.4 错误处理模式

**退出码规范**

- **0**: 成功
- **1**: 一般错误
- **2**: 参数错误
- **3**: 计算错误
- **127**: 命令不存在

**错误信息格式**

```json
{
  "error": {
    "code": "INVALID_DATE",
    "message": "日期格式无效",
    "details": "期望格式: YYYY-MM-DD"
  }
}
```

---

## 三、核心设计模式

### 3.1 命令模式

```typescript
// TypeScript示例 (qimen-cli)
interface Command {
  name: string;
  description: string;
  execute(args: any): Promise<void>;
}

class CalcCommand implements Command {
  name = 'calc';
  description = '计算奇门盘';
  
  async execute(args: any): Promise<void> {
    const chart = await qimenEngine.calculate(args);
    console.log(formatter.format(chart, args.format));
  }
}
```

```go
// Go示例 (bazi-go)
type Command interface {
    Name() string
    Execute(args []string) error
}

type CalcCommand struct{}

func (c *CalcCommand) Name() string {
    return "calc"
}

func (c *CalcCommand) Execute(args []string) error {
    chart := bazi.Calculate(args)
    fmt.Println(chart)
    return nil
}
```

### 3.2 策略模式

```python
# Python示例 (fengshui-python)
class OutputFormatter:
    def format(self, data):
        raise NotImplementedError

class TableFormatter(OutputFormatter):
    def format(self, data):
        # 表格格式化
        pass

class JsonFormatter(OutputFormatter):
    def format(self, data):
        return json.dumps(data)

class FormatterFactory:
    formatters = {
        'table': TableFormatter,
        'json': JsonFormatter,
        'csv': CsvFormatter,
    }
    
    @classmethod
    def create(cls, format_type):
        return cls.formatters.get(format_type, TableFormatter)()
```

### 3.3 工厂模式

```rust
// Rust示例 (lunar-rust)
trait Calculator {
    fn calculate(&self, input: &str) -> Result<String, Error>;
}

struct LunarCalculator;
struct SolarCalculator;
struct GanzhiCalculator;

impl Calculator for LunarCalculator {
    fn calculate(&self, input: &str) -> Result<String, Error> {
        // 农历计算
        Ok(result)
    }
}

struct CalculatorFactory;

impl CalculatorFactory {
    fn create(calc_type: &str) -> Box<dyn Calculator> {
        match calc_type {
            "lunar" => Box::new(LunarCalculator),
            "solar" => Box::new(SolarCalculator),
            "ganzhi" => Box::new(GanzhiCalculator),
            _ => panic!("Unknown calculator type"),
        }
    }
}
```

---

## 四、统一设计规范

### 4.1 命令命名规范

**动词+名词结构**

- calc: 计算
- show: 显示
- list: 列表
- export: 导出
- import: 导入
- config: 配置

**玄学专用命令**

- pai: 排盘
- qi: 起局
- fei: 飞星
- bu: 布盘

### 4.2 参数命名规范

**通用参数**

- --date / -d: 日期
- --time / -t: 时间
- --format / -f: 输出格式
- --output / -o: 输出文件
- --config / -c: 配置文件
- --verbose / -v: 详细输出
- --quiet / -q: 静默模式

**玄学专用参数**

- --longitude: 经度
- --latitude: 纬度
- --gender: 性别
- --method: 排盘方法
- --school: 流派

### 4.3 输出格式规范

**JSON格式标准**

```json
{
  "meta": {
    "version": "1.0",
    "timestamp": "2024-01-01T12:00:00Z",
    "command": "calc"
  },
  "data": {
    // 具体数据
  },
  "error": null
}
```

**错误格式标准**

```json
{
  "meta": {
    "version": "1.0",
    "timestamp": "2024-01-01T12:00:00Z"
  },
  "data": null,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "参数无效",
    "details": "日期格式应为YYYY-MM-DD"
  }
}
```

---

## 五、性能对比

### 5.1 启动速度

- **Rust**: 最快 (<10ms)
- **Go**: 很快 (<50ms)
- **Node.js**: 中等 (100-300ms)
- **Python**: 较慢 (200-500ms)

### 5.2 计算性能

- **Rust**: 最优 (原生性能)
- **Go**: 优秀 (接近原生)
- **Node.js**: 良好 (V8优化)
- **Python**: 一般 (解释执行)

### 5.3 内存占用

- **Rust**: 最低 (<5MB)
- **Go**: 较低 (<10MB)
- **Node.js**: 中等 (20-50MB)
- **Python**: 较高 (30-80MB)

---

## 六、推荐方案

### 6.1 按场景推荐

**高性能计算**

- **推荐**: Rust 或 Go
- **理由**: 性能最优

**快速开发**

- **推荐**: Node.js 或 Python
- **理由**: 开发效率高

**企业级应用**

- **推荐**: Go
- **理由**: 部署简单，性能优秀

### 6.2 统一CLI框架建议

建议开发统一的跨语言CLI框架:

```yaml
# 命令定义文件
name: xuanxue-cli
version: 1.0.0
commands:
  calc:
    description: 计算排盘
    args:
      - name: date
        type: string
        required: true
      - name: format
        type: string
        default: table
    output:
      formats: [table, json, csv]
```

---

## 七、结论

### 7.1 设计原则

- **一致性**: 命令结构、参数命名保持一致
- **可扩展性**: 易于添加新功能
- **可测试性**: 便于单元测试
- **文档化**: 完善的帮助信息

### 7.2 未来趋势

- **标准化**: CLI接口标准化
- **组合化**: 工具链组合使用
- **智能化**: AI辅助命令补全

---

**分析报告完成**  
**分析人员**: 集群E Agent  
**报告版本**: v1.0
