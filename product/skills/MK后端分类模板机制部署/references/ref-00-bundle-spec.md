# 包导入规范（ref-04）

> 本文档集中声明所有跨模块包的标准导入。**ref-01、ref-02、ref-03 均须遵守本规范**。
> 各 ref 只需在代码块中保留本机制新建类的导入；所有来自外部模块的类，以本文档为准。

---

## 一、JDK / 标准库

```java
import javax.persistence.*;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import javax.validation.constraints.*;
import java.util.*;
import java.util.Arrays;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.stream.Collectors;
```

---

## 二、Lombok

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import lombok.extern.slf4j.Slf4j;
```

---

## 三、Hutool

```java
import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.comparator.CompareUtil;
import cn.hutool.core.util.StrUtil;
```

---

## 四、Spring Framework

```java
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.CollectionUtils;
import org.springframework.web.bind.annotation.*;
```

---

## 五、Swagger

```java
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
```

---

## 六、Landray Common（框架基础层）

```java
// 基础实体 / VO
import com.landray.common.core.entity.AbstractEntity;
import com.landray.common.core.entity.TreeEntity;
import com.landray.common.core.dto.AbstractVO;
import com.landray.common.core.dto.IdVO;
import com.landray.common.core.dto.IdNameProperty;
import com.landray.common.core.dto.QueryRequest;
import com.landray.common.core.dto.QueryResult;
import com.landray.common.core.dto.Response;

// 字段接口
import com.landray.common.core.data.field.FdAlterTime;
import com.landray.common.core.data.field.FdCreateTime;
import com.landray.common.core.data.field.FdLastModifiedTime;
import com.landray.common.core.data.field.FdName4Language;
import com.landray.common.core.data.field.FdOrder;
import com.landray.common.core.data.field.FdSubject;
import com.landray.common.core.data.field.FdDelete;


// 枚举 / 常量
import com.landray.common.core.constant.IEnum;
import com.landray.common.core.constant.QueryConstant;
import com.landray.common.core.constant.QueryConstant.Operator;

// Repository / Service 基类
import com.landray.common.core.repository.IRepository;
import com.landray.common.core.service.AbstractServiceImpl;

// 元数据注解
import com.landray.common.core.annotation.MetaEntity;
import com.landray.common.core.annotation.MetaProperty;

// ID 生成
import com.landray.common.core.util.IDGenerator;

// 异常
import com.landray.common.exception.KmssRuntimeException;
```

---

## 七、Landray Sys Auth（权限层）

```java
// 权限字段接口
import com.landray.sys.auth.data.field.FdAlter;
import com.landray.sys.auth.data.field.FdCreator;
import com.landray.sys.auth.data.field.FdCreatorDept;
import com.landray.sys.auth.data.field.FdOwner;
import com.landray.sys.auth.data.field.FdOwnerDept;

// 权限实体
import com.landray.sys.auth.data.entity.PermissionInfoEntity;
```

---

## 八、Landray Sys Category（分类机制）

```java
// 分类实体接口
import com.landray.sys.category.entity.CategoryEntity;

// 分类服务接口
import com.landray.sys.category.api.ICategoryApi;
import com.landray.sys.category.api.ICategoryService;

// 分类 VO / DTO
import com.landray.sys.category.dto.AbstractCategoryVO;
import com.landray.sys.category.dto.TreeNodeVO;
import com.landray.sys.category.dto.TreeParamDTO;
import com.landray.sys.category.dto.TreeRequestVO;

// 分类常量
import com.landray.sys.category.constant.NodeType;

// 分类 Controller 接口
import com.landray.sys.category.controller.CategoryController;
```

---

## 九、Landray Framework Support

```java
import com.landray.framework.support.ApplicationContextHolder;
```

---

## 十、Landray Web（MVC 基类）

```java
import com.landray.web.controller.AbstractController;
import com.landray.web.controller.CombineController;
import com.landray.web.controller.PreloadController;
import com.landray.web.api.IApi;
```

---

## 十一、Landray Sys Xform（表单工具）

```java
import com.landray.sys.xform.util.BeanHelper;
```

---

## 十二、Jackson / FastJSON

```java
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.annotation.JSONField;
import com.fasterxml.jackson.annotation.JsonIgnore;
```
