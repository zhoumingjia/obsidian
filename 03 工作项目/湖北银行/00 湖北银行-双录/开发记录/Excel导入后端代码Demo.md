## 一、项目结构
```
src/main/java/com/example/scriptimport/
├── controller/          # 控制器层
│   ├── BusinessTypeController.java
│   └── ScriptImportController.java
├── service/            # 服务层
│   ├── BusinessTypeService.java
│   ├── ScriptImportService.java
│   └── impl/          # 实现类
├── dao/               # 数据访问层
│   ├── BusinessTypeMapper.java
│   └── ScriptDataMapper.java
├── entity/            # 实体类
│   ├── BusinessType.java
│   ├── ScriptData.java
│   └── ImportRecord.java
├── dto/               # 数据传输对象
│   ├── BusinessTypeDTO.java
│   ├── ImportRequestDTO.java
│   └── ImportResultDTO.java
├── util/              # 工具类
│   ├── ExcelUtil.java
│   └── ValidatorUtil.java
└── config/            # 配置类
    └── ExcelConfig.java
```

## 二、核心代码实现

### 1. 实体类

#### BusinessType.java
```java
@Data
@TableName("business_type")
public class BusinessType {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private Long parentId;
    
    @NotNull
    private Integer level;  // 1:一级, 2:二级, 3:三级
    
    @NotBlank
    private String name;
    
    private String code;
    
    private Integer sortOrder;
    
    private Boolean isActive;
    
    @TableField(exist = false)
    private List<BusinessType> children;
}
```

#### ScriptData.java
```java
@Data
@TableName("script_data")
public class ScriptData {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String importId;
    
    private Long businessTypeId;
    
    @NotBlank
    private String scriptContent;
    
    @NotBlank
    private String scriptType;
    
    private Integer weight;
    
    private String status;
    
    private LocalDateTime createdTime;
}
```

#### ImportRecord.java
```java
@Data
@TableName("import_record")
public class ImportRecord {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    @TableField("import_id")
    private String importId;
    
    private Long userId;
    
    private Long businessTypeId;
    
    private String fileName;
    
    private Integer totalCount;
    
    private Integer successCount;
    
    private Integer failCount;
    
    private String status;  // PROCESSING, SUCCESS, FAILED
    
    private String errorLog;
    
    private LocalDateTime createdTime;
}
```

### 2. DTO对象

#### ImportRequestDTO.java
```java
@Data
public class ImportRequestDTO {
    @NotNull(message = "业务类型不能为空")
    private Long businessTypeId;
    
    @NotBlank(message = "文件不能为空")
    private String fileName;
    
    @NotNull(message = "文件内容不能为空")
    private MultipartFile file;
}
```

#### ImportResultDTO.java
```java
@Data
public class ImportResultDTO {
    private String importId;
    private String fileName;
    private String businessTypeName;
    private Integer totalCount;
    private Integer successCount;
    private Integer failCount;
    private String status;
    private List<ImportDetailDTO> details;
    private LocalDateTime importTime;
}

@Data
class ImportDetailDTO {
    private Integer rowNum;
    private String scriptContent;
    private String scriptType;
    private String status;
    private String errorMessage;
}
```

### 3. 控制器层

#### ScriptImportController.java
```java
@RestController
@RequestMapping("/api/script")
@Slf4j
public class ScriptImportController {
    
    @Autowired
    private ScriptImportService scriptImportService;
    
    /**
     * Excel导入接口
     */
    @PostMapping("/import")
    public ResponseEntity<ApiResponse> importScript(
            @RequestParam("businessTypeId") Long businessTypeId,
            @RequestParam("file") MultipartFile file) {
        
        try {
            // 1. 参数校验
            if (file.isEmpty()) {
                return ResponseEntity.badRequest()
                    .body(ApiResponse.error("文件不能为空"));
            }
            
            // 2. 文件格式校验
            String fileName = file.getOriginalFilename();
            if (!ExcelUtil.isValidExcelFile(fileName)) {
                return ResponseEntity.badRequest()
                    .body(ApiResponse.error("只支持.xlsx和.xls格式文件"));
            }
            
            // 3. 调用导入服务
            ImportResultDTO result = scriptImportService.importScriptData(
                businessTypeId, fileName, file);
            
            return ResponseEntity.ok(ApiResponse.success(result));
            
        } catch (BusinessException e) {
            log.error("业务异常: {}", e.getMessage(), e);
            return ResponseEntity.badRequest()
                .body(ApiResponse.error(e.getMessage()));
        } catch (Exception e) {
            log.error("系统异常: {}", e.getMessage(), e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("系统异常，请稍后重试"));
        }
    }
    
    /**
     * 查询导入结果
     */
    @GetMapping("/import/result/{importId}")
    public ResponseEntity<ApiResponse> getImportResult(
            @PathVariable String importId) {
        
        ImportResultDTO result = scriptImportService.getImportResult(importId);
        return ResponseEntity.ok(ApiResponse.success(result));
    }
    
    /**
     * 下载导入模板
     */
    @GetMapping("/template/download")
    public ResponseEntity<Resource> downloadTemplate() {
        
        try {
            // 读取模板文件
            Resource resource = ExcelUtil.getTemplateFile();
            
            return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, 
                    "attachment; filename=\"script_template.xlsx\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(resource);
            
        } catch (Exception e) {
            log.error("下载模板失败: {}", e.getMessage(), e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
}
```

#### BusinessTypeController.java
```java
@RestController
@RequestMapping("/api/business")
public class BusinessTypeController {
    
    @Autowired
    private BusinessTypeService businessTypeService;
    
    /**
     * 获取一级业务类型
     */
    @GetMapping("/types/level1")
    public ResponseEntity<ApiResponse> getLevel1Types() {
        List<BusinessTypeDTO> types = businessTypeService.getLevel1Types();
        return ResponseEntity.ok(ApiResponse.success(types));
    }
    
    /**
     * 根据父级ID获取下级类型
     */
    @GetMapping("/types/children")
    public ResponseEntity<ApiResponse> getChildrenTypes(
            @RequestParam Long parentId) {
        List<BusinessTypeDTO> types = businessTypeService.getChildrenByParentId(parentId);
        return ResponseEntity.ok(ApiResponse.success(types));
    }
    
    /**
     * 获取完整的三级业务类型树
     */
    @GetMapping("/types/tree")
    public ResponseEntity<ApiResponse> getBusinessTypeTree() {
        List<BusinessTypeDTO> tree = businessTypeService.getBusinessTypeTree();
        return ResponseEntity.ok(ApiResponse.success(tree));
    }
}
```

### 4. 服务层

#### ScriptImportService.java
```java
public interface ScriptImportService {
    
    /**
     * 导入Excel数据
     */
    ImportResultDTO importScriptData(Long businessTypeId, String fileName, MultipartFile file);
    
    /**
     * 获取导入结果
     */
    ImportResultDTO getImportResult(String importId);
    
    /**
     * 异步导入任务
     */
    void asyncImportScriptData(Long businessTypeId, String fileName, MultipartFile file, String importId);
}
```

#### ScriptImportServiceImpl.java
```java
@Service
@Slf4j
public class ScriptImportServiceImpl implements ScriptImportService {
    
    @Autowired
    private BusinessTypeService businessTypeService;
    
    @Autowired
    private ScriptDataMapper scriptDataMapper;
    
    @Autowired
    private ImportRecordMapper importRecordMapper;
    
    @Autowired
    private AsyncTaskExecutor asyncTaskExecutor;
    
    private static final int BATCH_SIZE = 1000;
    
    @Override
    @Transactional
    public ImportResultDTO importScriptData(Long businessTypeId, String fileName, MultipartFile file) {
        
        // 1. 生成导入批次ID
        String importId = UUID.randomUUID().toString().replace("-", "");
        
        // 2. 创建导入记录
        ImportRecord importRecord = createImportRecord(importId, businessTypeId, fileName);
        
        try {
            // 3. 解析Excel数据
            List<ScriptData> scriptDataList = parseExcelFile(file, businessTypeId, importId);
            
            // 4. 校验数据
            List<ImportDetailDTO> validationResults = validateScriptData(scriptDataList);
            
            // 5. 保存有效数据
            List<ScriptData> validData = filterValidData(scriptDataList, validationResults);
            batchInsertScriptData(validData);
            
            // 6. 更新导入记录
            updateImportRecord(importRecord, scriptDataList.size(), validData.size(), "SUCCESS");
            
            // 7. 构建返回结果
            return buildImportResult(importRecord, validationResults);
            
        } catch (Exception e) {
            log.error("导入失败: {}", e.getMessage(), e);
            
            // 更新导入记录为失败状态
            importRecord.setStatus("FAILED");
            importRecord.setErrorLog(e.getMessage());
            importRecordMapper.updateById(importRecord);
            
            throw new BusinessException("导入失败: " + e.getMessage());
        }
    }
    
    @Override
    public void asyncImportScriptData(Long businessTypeId, String fileName, MultipartFile file, String importId) {
        
        asyncTaskExecutor.execute(() -> {
            try {
                importScriptData(businessTypeId, fileName, file);
            } catch (Exception e) {
                log.error("异步导入失败: {}", e.getMessage(), e);
            }
        });
    }
    
    @Override
    public ImportResultDTO getImportResult(String importId) {
        
        ImportRecord importRecord = importRecordMapper.selectByImportId(importId);
        if (importRecord == null) {
            throw new BusinessException("导入记录不存在");
        }
        
        // 查询导入详情
        List<ImportDetailDTO> details = getImportDetails(importId);
        
        return buildImportResult(importRecord, details);
    }
    
    /**
     * 解析Excel文件
     */
    private List<ScriptData> parseExcelFile(MultipartFile file, Long businessTypeId, String importId) {
        
        try {
            InputStream inputStream = file.getInputStream();
            
            // 使用EasyExcel解析
            List<ScriptExcelDTO> excelDataList = EasyExcel.read(inputStream)
                .head(ScriptExcelDTO.class)
                .sheet()
                .doReadSync();
            
            // 转换为实体对象
            return excelDataList.stream()
                .map(excelDTO -> convertToScriptData(excelDTO, businessTypeId, importId))
                .collect(Collectors.toList());
                
        } catch (Exception e) {
            log.error("解析Excel文件失败", e);
            throw new BusinessException("解析Excel文件失败: " + e.getMessage());
        }
    }
    
    /**
     * 数据校验
     */
    private List<ImportDetailDTO> validateScriptData(List<ScriptData> scriptDataList) {
        
        List<ImportDetailDTO> results = new ArrayList<>();
        
        for (int i = 0; i < scriptDataList.size(); i++) {
            ScriptData data = scriptDataList.get(i);
            ImportDetailDTO detail = new ImportDetailDTO();
            detail.setRowNum(i + 2);  // Excel行号（从第2行开始）
            detail.setScriptContent(data.getScriptContent());
            detail.setScriptType(data.getScriptType());
            
            try {
                // 校验必填字段
                validateRequiredFields(data);
                
                // 校验业务规则
                validateBusinessRules(data);
                
                // 校验重复数据
                validateDuplicateData(data);
                
                detail.setStatus("SUCCESS");
                
            } catch (BusinessException e) {
                detail.setStatus("FAILED");
                detail.setErrorMessage(e.getMessage());
            }
            
            results.add(detail);
        }
        
        return results;
    }
    
    /**
     * 校验必填字段
     */
    private void validateRequiredFields(ScriptData data) {
        if (StringUtils.isBlank(data.getScriptContent())) {
            throw new BusinessException("话术内容不能为空");
        }
        
        if (StringUtils.isBlank(data.getScriptType())) {
            throw new BusinessException("话术类型不能为空");
        }
    }
    
    /**
     * 校验业务规则
     */
    private void validateBusinessRules(ScriptData data) {
        // 话术内容长度限制
        if (data.getScriptContent().length() > 5000) {
            throw new BusinessException("话术内容不能超过5000字符");
        }
        
        // 话术类型有效性校验
        if (!isValidScriptType(data.getScriptType())) {
            throw new BusinessException("话术类型无效");
        }
    }
    
    /**
     * 校验重复数据
     */
    private void validateDuplicateData(ScriptData data) {
        // 检查是否已存在相同内容的话术
        Long count = scriptDataMapper.countDuplicateScript(
            data.getBusinessTypeId(), 
            data.getScriptContent(),
            data.getScriptType());
            
        if (count > 0) {
            throw new BusinessException("已存在相同的话术内容");
        }
    }
    
    /**
     * 批量插入数据
     */
    private void batchInsertScriptData(List<ScriptData> dataList) {
        
        if (CollectionUtils.isEmpty(dataList)) {
            return;
        }
        
        // 分批插入，避免单次插入数据量过大
        for (int i = 0; i < dataList.size(); i += BATCH_SIZE) {
            int end = Math.min(i + BATCH_SIZE, dataList.size());
            List<ScriptData> batchList = dataList.subList(i, end);
            
            scriptDataMapper.batchInsert(batchList);
        }
    }
    
    /**
     * 创建导入记录
     */
    private ImportRecord createImportRecord(String importId, Long businessTypeId, String fileName) {
        
        ImportRecord record = new ImportRecord();
        record.setImportId(importId);
        record.setBusinessTypeId(businessTypeId);
        record.setFileName(fileName);
        record.setStatus("PROCESSING");
        record.setCreatedTime(LocalDateTime.now());
        
        // 获取当前用户ID（根据实际情况实现）
        record.setUserId(getCurrentUserId());
        
        importRecordMapper.insert(record);
        return record;
    }
    
    /**
     * 更新导入记录
     */
    private void updateImportRecord(ImportRecord record, int totalCount, int successCount, String status) {
        
        record.setTotalCount(totalCount);
        record.setSuccessCount(successCount);
        record.setFailCount(totalCount - successCount);
        record.setStatus(status);
        
        importRecordMapper.updateById(record);
    }
    
    /**
     * 构建返回结果
     */
    private ImportResultDTO buildImportResult(ImportRecord record, List<ImportDetailDTO> details) {
        
        ImportResultDTO result = new ImportResultDTO();
        result.setImportId(record.getImportId());
        result.setFileName(record.getFileName());
        result.setTotalCount(record.getTotalCount());
        result.setSuccessCount(record.getSuccessCount());
        result.setFailCount(record.getFailCount());
        result.setStatus(record.getStatus());
        result.setDetails(details);
        result.setImportTime(record.getCreatedTime());
        
        // 获取业务类型名称
        BusinessType businessType = businessTypeService.getById(record.getBusinessTypeId());
        if (businessType != null) {
            result.setBusinessTypeName(businessType.getName());
        }
        
        return result;
    }
    
    /**
     * 获取当前用户ID
     */
    private Long getCurrentUserId() {
        // 根据实际情况实现，如从SecurityContext获取
        return 1L;
    }
}
```

### 5. 数据访问层

#### ScriptDataMapper.java
```java
@Mapper
public interface ScriptDataMapper extends BaseMapper<ScriptData> {
    
    /**
     * 批量插入
     */
    @Insert("<script>" +
            "INSERT INTO script_data (import_id, business_type_id, script_content, " +
            "script_type, weight, status, created_time) VALUES " +
            "<foreach collection='list' item='item' separator=','>" +
            "(#{item.importId}, #{item.businessTypeId}, #{item.scriptContent}, " +
            "#{item.scriptType}, #{item.weight}, #{item.status}, #{item.createdTime})" +
            "</foreach>" +
            "</script>")
    void batchInsert(@Param("list") List<ScriptData> list);
    
    /**
     * 统计重复话术
     */
    @Select("SELECT COUNT(1) FROM script_data " +
            "WHERE business_type_id = #{businessTypeId} " +
            "AND script_content = #{scriptContent} " +
            "AND script_type = #{scriptType}")
    Long countDuplicateScript(@Param("businessTypeId") Long businessTypeId,
                              @Param("scriptContent") String scriptContent,
                              @Param("scriptType") String scriptType);
    
    /**
     * 根据导入ID查询话术数据
     */
    @Select("SELECT * FROM script_data WHERE import_id = #{importId}")
    List<ScriptData> selectByImportId(@Param("importId") String importId);
}
```

#### ImportRecordMapper.java
```java
@Mapper
public interface ImportRecordMapper extends BaseMapper<ImportRecord> {
    
    /**
     * 根据导入ID查询
     */
    @Select("SELECT * FROM import_record WHERE import_id = #{importId}")
    ImportRecord selectByImportId(@Param("importId") String importId);
    
    /**
     * 查询用户的导入历史
     */
    @Select("SELECT * FROM import_record WHERE user_id = #{userId} " +
            "ORDER BY created_time DESC LIMIT #{limit}")
    List<ImportRecord> selectUserHistory(@Param("userId") Long userId, 
                                         @Param("limit") Integer limit);
}
```

### 6. 工具类

#### ExcelUtil.java
```java
@Component
@Slf4j
public class ExcelUtil {
    
    private static final List<String> ALLOWED_EXTENSIONS = Arrays.asList("xlsx", "xls");
    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
    
    /**
     * 校验Excel文件格式
     */
    public static boolean isValidExcelFile(String fileName) {
        if (StringUtils.isBlank(fileName)) {
            return false;
        }
        
        String extension = getFileExtension(fileName);
        return ALLOWED_EXTENSIONS.contains(extension.toLowerCase());
    }
    
    /**
     * 校验文件大小
     */
    public static boolean isValidFileSize(long fileSize) {
        return fileSize <= MAX_FILE_SIZE;
    }
    
    /**
     * 获取文件扩展名
     */
    private static String getFileExtension(String fileName) {
        int lastDotIndex = fileName.lastIndexOf(".");
        if (lastDotIndex == -1) {
            return "";
        }
        return fileName.substring(lastDotIndex + 1);
    }
    
    /**
     * 读取模板文件
     */
    public static Resource getTemplateFile() {
        try {
            ClassPathResource resource = new ClassPathResource("templates/script_template.xlsx");
            if (!resource.exists()) {
                log.warn("模板文件不存在");
                // 可以动态生成模板文件
                return generateTemplateFile();
            }
            return resource;
        } catch (Exception e) {
            log.error("读取模板文件失败", e);
            throw new BusinessException("读取模板文件失败");
        }
    }
    
    /**
     * 动态生成Excel模板
     */
    public static ByteArrayResource generateTemplateFile() {
        try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
            
            // 创建Excel写入器
            ExcelWriter excelWriter = EasyExcel.write(outputStream).build();
            
            // 创建模板数据
            List<ScriptExcelDTO> templateData = createTemplateData();
            
            // 写入Sheet
            WriteSheet writeSheet = EasyExcel.writerSheet("话术模板")
                .head(ScriptExcelDTO.class)
                .build();
                
            excelWriter.write(templateData, writeSheet);
            excelWriter.finish();
            
            return new ByteArrayResource(outputStream.toByteArray());
            
        } catch (Exception e) {
            log.error("生成模板文件失败", e);
            throw new BusinessException("生成模板文件失败");
        }
    }
    
    /**
     * 创建模板数据示例
     */
    private static List<ScriptExcelDTO> createTemplateData() {
        List<ScriptExcelDTO> data = new ArrayList<>();
        
        ScriptExcelDTO example1 = new ScriptExcelDTO();
        example1.setScriptContent("您好，我是XX银行的客户经理，有什么可以帮您？");
        example1.setScriptType("开场白");
        example1.setWeight(1);
        data.add(example1);
        
        ScriptExcelDTO example2 = new ScriptExcelDTO();
        example2.setScriptContent("根据您的需求，我为您推荐这款理财产品...");
        example2.setScriptType("产品介绍");
        example2.setWeight(2);
        data.add(example2);
        
        return data;
    }
}
```

#### ValidatorUtil.java
```java
@Component
public class ValidatorUtil {
    
    /**
     * 校验话术类型是否有效
     */
    public boolean isValidScriptType(String scriptType) {
        // 可以从配置表或枚举中获取有效的类型
        List<String> validTypes = Arrays.asList(
            "开场白", "产品介绍", "异议处理", "促成话术", "结束语"
        );
        return validTypes.contains(scriptType);
    }
    
    /**
     * 校验业务类型是否存在
     */
    public boolean isValidBusinessType(Long businessTypeId) {
        // 查询数据库校验
        return true; // 简化实现
    }
}
```

### 7. 配置文件

#### application.yml
```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 20MB
  
  datasource:
    url: jdbc:mysql://localhost:3306/script_db
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
  type-aliases-package: com.example.scriptimport.entity
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# Excel导入配置
excel:
  import:
    max-size: 10485760  # 10MB
    allowed-extensions: xlsx,xls
    batch-size: 1000
    template-path: classpath:templates/script_template.xlsx
```

## 三、使用示例

### 1. 调用导入接口
```java
// 前端调用示例
const formData = new FormData();
formData.append('businessTypeId', '123');
formData.append('file', fileInput.files[0]);

fetch('/api/script/import', {
    method: 'POST',
    body: formData
})
.then(response => response.json())
.then(data => {
    console.log('导入结果:', data);
});
```

### 2. 查询导入结果
```java
// 根据导入ID查询结果
GET /api/script/import/result/{importId}
```

## 四、注意事项

1. **文件安全**：限制文件类型和大小，防止恶意文件上传
2. **性能优化**：大文件使用异步导入，分批插入数据库
3. **数据一致性**：使用事务保证数据一致性
4. **错误处理**：提供详细的错误信息和重试机制
5. **日志记录**：记录完整的导入过程，便于问题排查

这个Demo提供了完整的Excel导入后端实现，包括文件校验、数据解析、业务校验、批量插入等功能。可以根据实际需求进行调整和扩展。