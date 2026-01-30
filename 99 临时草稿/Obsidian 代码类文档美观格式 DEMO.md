（适配理财系统ESB接口整改类技术文档，轻量化、层次清、易阅读）
## 一、接口整改范围
- 理财系统核心ESB接口：`fund-api-01`（产品查询）、`fund-api-02`（申购提交）
- 整改类型：参数校验逻辑优化、返回值格式统一、超时机制新增
## 二、核心整改代码示例
### 2.1 参数校验逻辑（Java）
以下为`fund-api-01`接口入参非空/格式校验优化代码，新增正则校验与异常统一抛出：
```java
// 优化后参数校验方法
public void checkFundQueryParam(FundQueryParam param) {
    if (param == null) {
        throw new BizException("入参不能为空", "PARAM_NULL");
    }
    // 产品编码正则校验（6位数字）
    String fundCode = param.getFundCode();
    if (StringUtils.isBlank(fundCode) || !Pattern.matches("^\\d{6}$", fundCode)) {
        throw new BizException("产品编码为6位数字", "PARAM_FUND_CODE_ERROR");
    }
    // 渠道编码非空校验
    if (StringUtils.isBlank(param.getChannelCode())) {
        throw new BizException("渠道编码不能为空", "PARAM_CHANNEL_NULL");
    }
}
```

### 2.2 超时机制配置（YML）
ESB接口全局超时时间调整为3s，新增重试策略配置：
```yaml
esb:
  config:
    connect-timeout: 500  # 连接超时500ms
    read-timeout: 3000    # 读取超时3000ms
    retry:
      enable: true        # 开启重试
      count: 2            # 重试次数2次
      interval: 1000      # 重试间隔1000ms
```

## 三、投产实施步骤
1. **预发布环境部署**：将整改后代码包部署至预发布环境，执行`sh deploy-esb.sh fund`脚本
	1. dsfasd
	2. fsdfasf
	3. 232323
2. **接口联调测试**：调用预发布接口，验证参数校验、返回值、超时机制是否符合预期
3. **生产环境灰度**：先灰度5%流量至整改后接口，持续监控无报错后全量放开
4. **投产验证**：全量后执行核心用例回归，检查生产日志无异常则投产完成

## 四、关键注意事项
* 整改后接口**向下兼容**，旧版入参格式仍可正常调用，无迁移成本
	* fsadfasf
		* fdasfasf
	* 2323dsfsa
* 生产投产时间窗口：每日22:00-23:00（低峰期），禁止非窗口时段部署
* 异常回滚命令：`sh rollback-esb.sh fund v1.0`（回滚至整改前v1.0版本）

我可==以把这个==DEMO做成**Obsidian可直接复制的纯文本格式**，你粘贴到笔记里就能用，`需要`吗？


> [!info]
> Contents 阿斯弗撒打发士大夫

> [!example]
> Contents

> [!kanban]
> Contents

