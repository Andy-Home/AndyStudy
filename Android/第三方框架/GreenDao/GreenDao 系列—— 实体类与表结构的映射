&emsp;&emsp;当前的 *GreenDao* 最新版本为 *GreenDao3*，通过 **greendao-gradle-plugin** 能够将定义好的实体类(包含对应注解)自动生成对应的数据表结构，本篇文章不是对针对 **greendao-gradle-plugin** 分析，因为 **greendao-gradle-plugin** 没有开源😂，因此我们通过分析[已开源的生成实体类源码](https://github.com/greenrobot/greenDAO/tree/master/DaoGenerator)进行分析。

#### 整体框架
![image](https://raw.githubusercontent.com/Andy-Home/AndyStudy/master/Android/%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A1%86%E6%9E%B6/GreenDao/DaoGenerator%20%E4%B8%BB%E8%A6%81%E7%B1%BB%E7%AE%80%E8%A6%81%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

&emsp;&emsp;上图为DaoGenerator 项目中主要类的简单关系图，通过该图咱们能很清晰的看出他们之间的关系，我们首先从Schema类开始看。

#### Schema
&emsp;&emsp;**Schema** 可以理解为数据库，任何数据库都有一个唯一的 *Schema*。

``` java
public class Schema {
   
    ......
    
    /**
    * Entity实体类
    */
    private final List<Entity> entities;
    
    /**
    *   实体类属性与表格列属性的映射关系
    */
    private Map<PropertyType, String> propertyToDbType;
    private Map<PropertyType, String> propertyToJavaTypeNotNull;
    private Map<PropertyType, String> propertyToJavaTypeNullable;
    ......

    public Schema(String name, int version, String defaultJavaPackage) {
        ......
        initTypeMappings();
    }
    
    ......
    
    /**
    *   初始化表格列属性与实体类的映射关系
    */
    private void initTypeMappings() {
        propertyToDbType = new HashMap<>();
        propertyToDbType.put(PropertyType.Boolean, "INTEGER");
        ......
        propertyToDbType.put(PropertyType.Date, "INTEGER");

        propertyToJavaTypeNotNull = new HashMap<>();
        propertyToJavaTypeNotNull.put(PropertyType.Boolean, "boolean");
        ......
        propertyToJavaTypeNotNull.put(PropertyType.Date, "java.util.Date");

        propertyToJavaTypeNullable = new HashMap<>();
        propertyToJavaTypeNullable.put(PropertyType.Boolean, "Boolean");
        ......
        propertyToJavaTypeNullable.put(PropertyType.Date, "java.util.Date");
    }

    /**
    *   添加Entity的方法
    */
    public Entity addEntity(String className) {
        Entity entity = new Entity(this, className);
        entities.add(entity);
        return entity;
    }
    ......
}
```
&emsp;&emsp;上述代码为 *Schema* 的主要代码，在其中定义了*Entity* 的集合、实体类属性与数据库表格列属性的映射关系、以及增加 *Entity* 的方法。

&emsp;&emsp;将映射关系定义在 *Schema* 中说明所有的数据库表格列属性与实体类属性的互相转化都依赖于该类。但是他们是如何使用什么方法互相转化呢？
``` java
    /**
    * 根据PropertyType属性，获取数据库属性
    */
    public String mapToDbType(PropertyType propertyType) {
        return mapType(propertyToDbType, propertyType);
    }

   /**
    * 根据PropertyType属性，获取值为空时的Java属性
    */
    public String mapToJavaTypeNullable(PropertyType propertyType) {
        return mapType(propertyToJavaTypeNullable, propertyType);
    }

    /**
    * 根据PropertyType属性，获取值不为空时的Java属性
    */
    public String mapToJavaTypeNotNull(PropertyType propertyType) {
        return mapType(propertyToJavaTypeNotNull, propertyType);
    }

    /**
    * 根据PropertyType属性与map数组获取对应的映射关系
    */
    private String mapType(Map<PropertyType, String> map, PropertyType propertyType) {
        String dbType = map.get(propertyType);
        if (dbType == null) {
            throw new IllegalStateException("No mapping for " + propertyType);
        }
        return dbType;
    }
```

&emsp;&emsp;以上代码里面有一个重要的对象 **PropertyType** ，它作为数据库列属性与实体类属性的调和剂，实现了两者之间的映射关系，在  *initTypeMappings* 中初始化了对应的关系，该对象实际上是一个枚举对象，定义了当前的有效属性，源码如下：
```java
/**
 * Currently available types for properties.
 *
 * @author Markus
 */
public enum PropertyType {

    Byte(true), Short(true), Int(true), Long(true), Boolean(true), Float(true), Double(true),
    String(false), ByteArray(false), Date(false);

    private final boolean scalar;

    PropertyType(boolean scalar) {
        this.scalar = scalar;
    }

    /** True if the type can be prepresented using a scalar (primitive type). */
    public boolean isScalar() {
        return scalar;
    }
}
```
&emsp;&emsp;*PropertyType* 作为调和剂，说明其重要性，它应该贯穿整个项目始终，但是作为一个枚举对象有它的局限性，因此有了 **Property**

#### Property

&emsp;&emsp;**Property** 属性可以理解为数据库表格的列属性也能理解为实体类的属性，它通过 *Schema* 的方法实现两者之间的互相转化。看看源码：
```java
public class Property {

    public static class PropertyBuilder {
        private final Property property;

        public PropertyBuilder(Schema schema, Entity entity, PropertyType propertyType, String propertyName) {
            property = new Property(schema, entity, propertyType, propertyName);
        }
        
        ......
    }

    /**
    * Schema与Entity属性 它们与Property的关系为 1:n
    */
    private final Schema schema;
    private final Entity entity;
    
    /**
    * 作为实现表结构列属性与实体类之间的映射关系的重要属性
    */
    private PropertyType propertyType;
    
    /**
    * 对应实体类的属性的名字与类型
    */
    private final String propertyName;
    private String javaType;

    /**
    * 对应数据库的名字与类型
    */
    private String dbName;
    private String dbType;
    
    ......

    public Property(Schema schema, Entity entity, PropertyType propertyType, String propertyName) {
        this.schema = schema;
        this.entity = entity;
        this.propertyName = propertyName;
        this.propertyType = propertyType;
    }

    ......
    
    /**
    *   初始化数据库表格列属性与实体类属性
    */
    void init2ndPass() {
        ......
        if (dbType == null) {
            dbType = schema.mapToDbType(propertyType);
        }
        if (dbName == null) {
            dbName = DaoUtil.dbName(propertyName);
            nonDefaultDbName = false;
        } else if (primaryKey && propertyType == PropertyType.Long && dbName.equals("_id")) {
            nonDefaultDbName = false;
        }

        // For backwards compatibility, consider notNull. It should be only dependent on nonPrimitiveType in the future.
        if (notNull && !nonPrimitiveType) {
            javaType = schema.mapToJavaTypeNotNull(propertyType);
        } else {
            javaType = schema.mapToJavaTypeNullable(propertyType);
        }
    }
}
```
&emsp;&emsp;*Property* 当然不止这些代码，它还实现了数据库列属性的约束，排序等等的与数据库相关的属性，当前咱们只针对实体类与表结构的映射进行分析。

&emsp;&emsp;通过上述代码可以知道，*Property* 中定义了实体类属性的对象与对象名，还有数据库列属性的名称与类型，通过 *init2ndPass* 方法引用 *Schema* 中的方法实现两者之间的具象的映射关系。因此我们能够认为：**Property 替代了 PropertyType 的调和剂作用，弥补了枚举的局限性，所以 Property 才是实现实体类与表结构的最小对象。**

#### Entity

&emsp;&emsp;上面已经分析了 *Schema* 与 *Property* ，其中 *Schema* 与数据库对应，*Property* 与数据库列属性以及实体类属性对应，但是咱们还不知道实体类还有表结构和谁对应呢？这就是我们接下来要分析的 **Entity**。

```java
public class Entity {

    /**
    * 该类属于哪个Schema
    */
    private final Schema schema;
    
    /**
    * 实体类名
    */
    private final String className;
    
    /**
    * 所有属性的集合
    */
    private final List<Property> properties;
    
    /**
    * 主键相关的属性
    */
    private final List<Property> propertiesPk;
    
    /**
    * 非主键相关的属性
    */
    private final List<Property> propertiesNonPk;
    
    /**
    * 属性名称集合
    */
    private final Set<String> propertyNames;
    
    /**
    * 数据库名
    */
    private String dbName;
    
    /**
    * 实体类的Dao类名
    */
    private String classNameDao;
    ......

    Entity(Schema schema, String className) {
        this.schema = schema;
        this.className = className;
        properties = new ArrayList<>();
        propertiesPk = new ArrayList<>();
        propertiesNonPk = new ArrayList<>();
        propertyNames = new HashSet<>();
        ......
    }

    /**
    * addXXXProperty(String propertyName)方法相当于定义实体类的属性
    * 或者数据库列属性
    */
    public PropertyBuilder addBooleanProperty(String propertyName) {
        return addProperty(PropertyType.Boolean, propertyName);
    }

    public PropertyBuilder addByteProperty(String propertyName) {
        return addProperty(PropertyType.Byte, propertyName);
    }

    ......

    public PropertyBuilder addProperty(PropertyType propertyType, String propertyName) {
        if (!propertyNames.add(propertyName)) {
            throw new RuntimeException("Property already defined: " + propertyName);
        }
        PropertyBuilder builder = new PropertyBuilder(schema, this, propertyType, propertyName);
        properties.add(builder.getProperty());
        return builder;
    }

    /**
    * 主键id
    */
    public PropertyBuilder addIdProperty() {
        PropertyBuilder builder = addLongProperty("id");
        builder.dbName("_id").primaryKey();
        return builder;
    }
    
    ......

    public List<Property> getProperties() {
        return properties;
    }

    ......

    /**
    * 初始化 Property 列表的实体对象属性与数据库列属性的映射关系
    */
    void init2ndPass() {
        init2ndPassNamesWithDefaults();

        for (int i = 0; i < properties.size(); i++) {
            Property property = properties.get(i);
            property.setOrdinal(i);
            property.init2ndPass();
            if (property.isPrimaryKey()) {
                propertiesPk.add(property);
            } else {
                propertiesNonPk.add(property);
            }
        }

        ......
    }
    
    /**
    * 设置默认名字
    */
    protected void init2ndPassNamesWithDefaults() {
        if (dbName == null) {
            dbName = DaoUtil.dbName(className);
            nonDefaultDbName = false;
        }
        
        if (classNameDao == null) {
            classNameDao = className + "Dao";
        }

        ......
    }

    ......
}

```
```java
public class DaoUtil {
    public static String dbName(String javaName) {
        StringBuilder builder = new StringBuilder(javaName);
        for (int i = 1; i < builder.length(); i++) {
            boolean lastWasUpper = Character.isUpperCase(builder.charAt(i - 1));
            boolean isUpper = Character.isUpperCase(builder.charAt(i));
            if (isUpper && !lastWasUpper) {
                builder.insert(i, '_');
                i++;
            }
        }
        return builder.toString().toUpperCase();
    }
}
```
&emsp;&emsp;上述代码为部分代码，主要为了体现实体类与表结构的属性的映射关系，其中一些复杂的，例如数据库表之间的关联关系，以及数据库列属性的 unique、排序配置等等。

&emsp;&emsp;上述代码定义了**主键 Property**、**非主键 Property**，根据 *Property* 的功能，咱们就已经完整的知道了实体类的属性与属性名，数据库列属性与属性名，现在就差如何生成代码和表结构了啊。

#### DaoGenerator
&emsp;&emsp;说了这么多，咱们知道了实体类和表结构是怎么映射的，但是还是没有对应到具体的实体类和表结构啊，回到最初我们接触 *GreenDao* 的时候，最初 *GreenDao* 的使用方式如下：

```java
//定义Schema
Schema schema = new Schema(1, "org.greenrobot.testdao");
//定义Entity
Entity addressEntity = schema.addEntity("Addresse");
Property idProperty = addressEntity.addIdProperty().getProperty();
addressEntity.addIntProperty("count").index();
addressEntity.addIntProperty("dummy").notNull();

File outputDir = new File("build/test-out");
outputDir.mkdirs();

//生成实体类和表结构
new DaoGenerator().generateAll(schema, outputDir.getPath());
```
&emsp;&emsp;上面的代码简单的定义了 *Schema* 与 *Entity* 后，调用了**DaoGeneraator** 类的 **generateAll** 方法来生成对应的代码。
```java
public class DaoGenerator {

    /**
    * 生成代码的模版
    */
    private Template templateDao;
    private Template templateDaoMaster;
    private Template templateDaoSession;
    private Template templateEntity;
    private Template templateDaoUnitTest;
    private Template templateContentProvider;

    /**
    * 初始化模版
    */
    public DaoGenerator() throws IOException {
        ......
        
        Configuration config = getConfiguration("dao.ftl");
        templateDao = config.getTemplate("dao.ftl");
        templateDaoMaster = config.getTemplate("dao-master.ftl");
        templateDaoSession = config.getTemplate("dao-session.ftl");
        templateEntity = config.getTemplate("entity.ftl");
        templateDaoUnitTest = config.getTemplate("dao-unit-test.ftl");
        templateContentProvider = config.getTemplate("content-provider.ftl");
    }

    /**
    * 生成所有实体类相关代码以及数据库操作类
    */
    public void generateAll(Schema schema, String outDir, String outDirEntity, String outDirTest) throws Exception {
        long start = System.currentTimeMillis();

        File outDirFile = toFileForceExists(outDir);
        File outDirEntityFile = outDirEntity != null ? toFileForceExists(outDirEntity) : outDirFile;
        File outDirTestFile = outDirTest != null ? toFileForceExists(outDirTest) : null;

        schema.init2ndPass();
        schema.init3rdPass();

        System.out.println("Processing schema version " + schema.getVersion() + "...");

        List<Entity> entities = schema.getEntities();
        
        //根据Entity生成对应的实体类与Dao类
        for (Entity entity : entities) {
            generate(templateDao, outDirFile, entity.getJavaPackageDao(), entity.getClassNameDao(), schema, entity);
            if (!entity.isProtobuf() && !entity.isSkipGeneration()) {
                generate(templateEntity, outDirEntityFile, entity.getJavaPackage(), entity.getClassName(), schema, entity);
            }
            if (outDirTestFile != null && !entity.isSkipGenerationTest()) {
                String javaPackageTest = entity.getJavaPackageTest();
                String classNameTest = entity.getClassNameTest();
                File javaFilename = toJavaFilename(outDirTestFile, javaPackageTest, classNameTest);
                if (!javaFilename.exists()) {
                    generate(templateDaoUnitTest, outDirTestFile, javaPackageTest, classNameTest, schema, entity);
                } else {
                    System.out.println("Skipped " + javaFilename.getCanonicalPath());
                }
            }
            for (ContentProvider contentProvider : entity.getContentProviders()) {
                Map<String, Object> additionalObjectsForTemplate = new HashMap<>();
                additionalObjectsForTemplate.put("contentProvider", contentProvider);
                generate(templateContentProvider, outDirFile, entity.getJavaPackage(), entity.getClassName()
                        + "ContentProvider", schema, entity, additionalObjectsForTemplate);
            }
        }
        generate(templateDaoMaster, outDirFile, schema.getDefaultJavaPackageDao(),
                schema.getPrefix() + "DaoMaster", schema, null);
        generate(templateDaoSession, outDirFile, schema.getDefaultJavaPackageDao(),
                schema.getPrefix() + "DaoSession", schema, null);

        long time = System.currentTimeMillis() - start;
        System.out.println("Processed " + entities.size() + " entities in " + time + "ms");
    }

    /**
    * 具体生成操作
    */
    private void generate(Template template, File outDirFile, String javaPackage, String javaClassName, Schema schema,
                          Entity entity, Map<String, Object> additionalObjectsForTemplate) throws Exception {
        Map<String, Object> root = new HashMap<>();
        root.put("schema", schema);
        root.put("entity", entity);
        if (additionalObjectsForTemplate != null) {
            root.putAll(additionalObjectsForTemplate);
        }
        try {
            File file = toJavaFilename(outDirFile, javaPackage, javaClassName);
            //noinspection ResultOfMethodCallIgnored
            file.getParentFile().mkdirs();

            if (entity != null && entity.getHasKeepSections()) {
                checkKeepSections(file, root);
            }

            Writer writer = new FileWriter(file);
            try {
                template.process(root, writer);
                writer.flush();
                System.out.println("Written " + file.getCanonicalPath());
            } finally {
                writer.close();
            }
        } catch (Exception ex) {
            System.err.println("Data map for template: " + root);
            System.err.println("Error while generating " + javaPackage + "." + javaClassName + " ("
                    + outDirFile.getCanonicalPath() + ")");
            throw ex;
        }
    }
}

```
&emsp;&emsp;上述代码为根据已经定义好的 *Entity* 与 *Schema* 生成对应的代码，有固定的模版，我们能够看到声明引用的类为 **Template**，这里用到的技术为 **FreeMarker**，为一个模版引擎，定义的模版有它自己的专属语言，文件后缀名为ftl。我们直接分析实体类生成以及生成数据库框架的模版：
```ftl
public class ${entity.classNameDao} extends AbstractDao<${entity.className}, ${entity.pkType}> {

    public static final String TABLENAME = "${entity.dbName}";

    ......

<#if !entity.skipCreationInDb>
    /** Creates the underlying database table. */
    public static void createTable(Database db, boolean ifNotExists) {
        String constraint = ifNotExists? "IF NOT EXISTS ": "";
        db.execSQL("CREATE TABLE " + constraint + "\"${entity.dbName}\" (" + //
<#list entity.propertiesColumns as property>
                "\"${property.dbName}\" ${property.dbType}<#if property.constraints??> ${property.constraints} </#if><#if property_has_next>," +<#else>);");</#if> // ${property_index}: ${property.propertyName}
</#list>
<#if entity.indexes?has_content >
        // Add Indexes
<#list entity.indexes as index>
        db.execSQL("CREATE <#if index.unique>UNIQUE </#if>INDEX " + constraint + "${index.name} ON \"${entity.dbName}\"" +
                " (<#list index.properties 
as property>\"${property.dbName}\"<#if (index.propertiesOrder[property_index])??> ${index.propertiesOrder[property_index]}</#if><#sep>,</#list>);");
</#list>
</#if>         
    }

   ......
}
```
&emsp;&emsp;该代码为 *dao.ftl* 文件中的内容，读取的内容为 *Entity* 类中的值，根据该文件生成的模版名称也就是平常我们看到的 *XXXDao* 类，生成的具体名称参考的是 *Entity* 中的 *classNameDao* 属性。同样也自动生成了创建数据库表的命令，对应的列属性参考的值是 *Property* 中的 *dbName* 与 *dbType* 属性。
&emsp;&emsp;接下来分析实体类的生成：
```ftl
......
<#if entity.skipCreationInDb><#assign entityAttrs = entityAttrs + ["createInDb = false"]></#if>
@Entity<#if (entityAttrs?size > 0)>(${entityAttrs?join(", ")})</#if>
public class ${entity.className}<#if
entity.superclass?has_content> extends ${entity.superclass} </#if><#if
entity.interfacesToImplement?has_content> implements <#list entity.interfacesToImplement
as ifc>${ifc}<#if ifc_has_next>, </#if></#list></#if> {
<#list entity.properties as property>
<#assign notNull = property.notNull && !primitiveTypes?seq_contains(property.javaTypeInEntity)>
<#if property.primaryKey||notNull||property.unique||property.index??||property.nonDefaultDbName||property.converter??>

</#if>
<#if property.javaDocField ??>
${property.javaDocField}
</#if>
<#if property.codeBeforeField ??>
    ${property.codeBeforeField}
</#if>
<#if property.primaryKey>
    @Id<#if property.autoincrement>(autoincrement = true)</#if>
</#if>
<#if property.nonDefaultDbName>
    @Property(nameInDb = "${property.dbName}")
</#if>
<#if property.converter??>
    @Convert(converter = ${property.converter}.class, columnType = ${property.javaType}.class)
</#if>
<#if notNull>
    @NotNull
</#if>
<#if property.unique>
    @Unique
</#if>
<#if ((property.index.nonDefaultName)!false) && (property.index.unique)!false>
    @Index(name = "${property.index.name}", unique = true)
<#elseif (property.index.nonDefaultName)!false>
    @Index(name = "${property.index.name}")
<#elseif (property.index.unique)!false>
    @Index(unique = true)
<#elseif property.index??>
    @Index
</#if>
    private ${property.javaTypeInEntity} ${property.propertyName};
</#list>

<#if entity.active>
    /** Used to resolve relations */
    @Generated
    private transient ${schema.prefix}DaoSession daoSession;

    /** Used for active entity operations. */
    @Generated
    private transient ${entity.classNameDao} myDao;
<#list entity.toOneRelations as toOne>

<#if toOne.useFkProperty>
    @ToOne(joinProperty = "${toOne.fkProperties[0].propertyName}")
    private ${toOne.targetEntity.className} ${toOne.name};

    @Generated
    private transient ${toOne.resolvedKeyJavaType[0]} ${toOne.name}__resolvedKey;
<#else>
    @ToOne
<#if toOne.fkProperties[0].nonDefaultDbName>
    @Property(nameInDb = "${toOne.fkProperties[0].dbName}")
</#if>
<#if toOne.fkProperties[0].unique>
    @Unique
</#if>
<#if toOne.fkProperties[0].notNull>
    @NotNull
</#if>
    private ${toOne.targetEntity.className} ${toOne.name};

    @Generated
    private transient boolean ${toOne.name}__refreshed;
</#if>
</#list>
<#list entity.toManyRelations as toMany>

<#if toMany.sourceProperties??>
    @ToMany(joinProperties = {
<#list toMany.sourceProperties as sourceProperty>
        @JoinProperty(name = "${sourceProperty.propertyName}", referencedName = "${toMany.targetProperties[sourceProperty_index].propertyName}")<#sep>,
</#list>

    })
<#elseif toMany.targetProperties??>
    @ToMany(mappedBy = "${toMany.targetProperties[0]}")
<#else>
    @ToMany
    @JoinEntity(entity = ${toMany.joinEntity.className}.class, sourceProperty = "${toMany.sourceProperty.propertyName}", targetProperty = "${toMany.targetProperty.propertyName}")
</#if>
<#assign orderSpec = (toMany.orderSpec)!"0">
<#if orderSpec != "0">
    @OrderBy("${orderSpec}")
</#if>
    private List<${toMany.targetEntity.className}> ${toMany.name};
</#list>

</#if>
<#if entity.hasKeepSections>
    // KEEP FIELDS - put your custom fields here
${keepFields!}    // KEEP FIELDS END

</#if>
<#if entity.constructors>
    @Generated
    public ${entity.className}() {
    }
<#if entity.propertiesPk?has_content && entity.propertiesPk?size != entity.properties?size>

    public ${entity.className}(<#list entity.propertiesPk as
property>${property.javaType} ${property.propertyName}<#if property_has_next>, </#if></#list>) {
<#list entity.propertiesPk as property>
        this.${property.propertyName} = ${property.propertyName};
</#list>
    }
</#if>

    @Generated
    public ${entity.className}(<#list entity.properties as
property>${property.javaTypeInEntity} ${property.propertyName}<#if property_has_next>, </#if></#list>) {
<#list entity.properties as property>
        this.${property.propertyName} = ${property.propertyName};
</#list>
    }
</#if>

<#if entity.active>
    /** called by internal mechanisms, do not call yourself. */
    @Generated
    public void __setDaoSession(${schema.prefix}DaoSession daoSession) {
        this.daoSession = daoSession;
        myDao = daoSession != null ? daoSession.get${entity.classNameDao?cap_first}() : null;
    }

</#if>
<#list entity.properties as property>
<#if property.javaDocGetter ??>
${property.javaDocGetter}
</#if>
<#if property.codeBeforeGetter ??>
    ${property.codeBeforeGetter}
</#if>
<#if property.notNull && !primitiveTypes?seq_contains(property.javaTypeInEntity)>
    @NotNull
</#if>
    public ${property.javaTypeInEntity} get${property.propertyName?cap_first}() {
        return ${property.propertyName};
    }

<#if property.notNull && !primitiveTypes?seq_contains(property.javaTypeInEntity)>
    /** Not-null value; ensure this value is available before it is saved to the database. */
</#if>
<#if property.javaDocSetter ??>
${property.javaDocSetter}
</#if>
<#if property.codeBeforeSetter ??>
    ${property.codeBeforeSetter}
</#if>
    public void set${property.propertyName?cap_first}(<#if property.notNull && !primitiveTypes?seq_contains(property.javaTypeInEntity)>@NotNull </#if>${property.javaTypeInEntity} ${property.propertyName}) {
        this.${property.propertyName} = ${property.propertyName};
    }

</#list>
    ......
}
```
&emsp;&emsp;该代码为 *entity.ftl* 文件中内容，使用了 *Property* 的 *propertyName* 属性，其中我们能够看到有一个*javaTypeInEntity* 属性，查找 *Property* 没有找到该属性，但是却有 *getJavaTypeInEntity()* 方法,查找资料发现，*FreeMark* 获取属性值的方法是通过 *get 方法* 获取，获取时他们将 get 省去了，这个需要注意。我们查看对应的方法。
```java
    public String getJavaTypeInEntity() {
        if (customTypeClassName != null) {
            return customTypeClassName;
        } else {
            return javaType;
        }
    }
```
实际获取到的属性为 *javaType*，或者属性 *customTypeClassName*，该属性为自定义的类的设置。最后生成的类与GreenDao3中咱们自己定义的类相似，包括注解。

#### 总结
&emsp;&emsp;上述分析，从对应数据库的 *Schema* 到对应实体类属性与数据库列属性的 *Property*，再到对应表与实体类的 *Entity*，根据这些分析了解实体类与表结构的映射关系是通过 *Map* 结构以及 *PropertyType* 作为中间键，实现数据库列属性与实体类属性的映射关系，并且通过 **FreeMarker** 技术、获取 *Entity* 与 *Property* 中的值自动生成对应的创建表结构的 **SQL命令** 以及实体类。