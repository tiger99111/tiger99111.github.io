---
layout: post
title: "linux下devicetree中常用的of函数"
date: 2017-08-25 
description: "devicetree"
tag: 博客 
---   

###linux下devicetree中常用的of函数

```
//从device_node中获取信息：
int of_property_read_u8_array(const struct device_node *np, const char *propname,u8 *out_values, size_t sz);
int of_property_read_u16_array(const struct device_node *np, const char *propname,u16 *out_values, size_t sz);
int of_property_read_u32_array(const struct device_node *np, const char *propname,u32 *out_values, size_t sz);
//从设备结点np中读取属性名为propname，类型为8、16、32、位整型数组的属性值，并放入out_values，sz指明了要读取的个数。

static inline int of_property_read_u8(const struct device_node *np,const char *propname,u8 *out_value) 
static inline int of_property_read_u16(const struct device_node *np,const char *propname,u8 *out_value) 
static inline int of_property_read_u32(const struct device_node *np,const char *propname,u8 *out_value) 
//从设备结点np中读取属性名为propname，类型为8、16、32位的属性值，并放入out_values。实际上这里调用的就是sz为1的XXX_array函数。
	 
int of_property_read_u32_index(const struct device_node *np,const char*propname,u32 index, u32 *out_value)
//从设备结点np中读取属性名为propname的属性值中第index个u32数值给out_value
	 
int of_property_read_u64(conststruct device_node *np, const char *propname,u64 *out_value)
//从设备结点np中读取属性名为propname，类型为64位的属性值，并放入out_values
 
int of_property_read_string(struct device_node *np, const char *propname,const char**out_string)
//从设备结点np中读取属性名为propname的字符串型属性值
 
int of_property_read_string_index(struct device_node *np, const char *propname,intindex, const char **output)
//从设备结点np中读取属性名为propname的字符串型属性值数组中的第index个字符串
 
int of_property_count_strings(struct device_node *np, const char *propname)
//从设备结点np中读取属性名为propname的字符串型属性值的个数
 
unsigned int irq_of_parse_and_map(struct device_node *dev, int index)
//从设备节点dev中读取第index个irq号
 
int of_irq_to_resource(struct device_node *dev, int index, struct resource *r)
//从设备节点dev中读取第index个irq号，并填充一个irq资源结构体
 
int of_irq_count(struct device_node *dev)
//获取设备节点dev的irq个数

static inline bool of_property_read_bool(const struct device_node *np,const char *propname);
//如果设备结点np含有propname属性，则返回true，否则返回false。一般用于检查空属性是否存在。
 
struct property* of_find_property(const struct device_node *np,const char *name,int *lenp)
//根据name参数，在指定的设备结点np中查找匹配的property，并返回这个property
 
const void * of_get_property(const struct device_node *np, const char *name,int *lenp)
//根据name参数，在指定的设备结点np中查找匹配的property，并返回这个property的属性值

struct device_node* of_get_parent(const struct device_node *node)
//获得node节点的父节点的device node

int of_device_is_compatible(const struct device_node *device,const char *compat);
//判断设备结点device的compatible属性是否包含compat指定的字符串

//从of_allnodes中查找信息：
struct device_node* of_find_node_by_path(const char *path)
//根据路径参数，在全局链表of_allnodes中，查找匹配的device_node

struct device_node* of_find_node_by_name(struct device_node *from,const char *name)
//则根据name在全局链表of_allnodes中查找匹配的device_node,若from=NULL表示从头开始查找

struct device_node* of_find_node_by_type(struct device_node *from,const char *type)
//根据设备类型在全局链表of_allnodes中查找匹配的device_node

struct device_node * of_find_compatible_node(struct device_node *from, const char*type, const char，*compatible);
//根据compatible的属性值在全局链表of_allnodes中查找匹配的device_node，大多数情况下，from、type为NULL。
 
struct device_node* of_find_node_with_property(struct device_node *from,const char *prop_name)
//根据节点属性的name在全局链表of_allnodes中查找匹配的device_node
 
struct device_node* of_find_node_by_phandle(phandle handle)
//根据phandle在全局链表of_allnodes中查找匹配的device_node
 
//杂：
void __iomem* of_iomap(struct device_node *node, int index);
//通过设备结点直接进行设备内存区间的 ioremap()，index是内存段的索引。若设备结点的reg属性有多段，可通过index标示要ioremap的是哪一段，只有1段的情况，index为0
 
unsigned long __init of_get_flat_dt_root(void)
//用来查找在dtb中的根节点，好像返回的都是0

int of_alias_get_id(struct device_node *np, const char *stem)
//获取节点np对应的aliasid号
 
struct device_node* of_node_get(struct device_node *node)
void of_node_put(struct device_node *node)
//device node计数增加/减少

const struct of_device_id* of_match_node(const struct of_device_id *matches,const struct device_node*node)
//将matches数组中of_device_id结构的name和type与device node的compatible和type匹配，返回匹配度最高的of_device_id结构

platform_device和resource相关：
int of_address_to_resource(struct device_node *dev, int index,struct resource *r)
//根据设备节点dev的reg属性值，填充资源结构体r。Index参数指明了使用reg属性中第几个属性值，一般设置为0，表示第一个。

struct platform_device* of_device_alloc(struct device_node *np,const char *bus_id,struct device *parent)
//根据device node，bus_id以及父节点创建该设备的platform_device结构，同时会初始化它的resource成员。
 
int of_platform_bus_probe(struct device_node *root,const struct of_device_id *matches,struct device *parent)
//遍历of_allnodes中的节点挂接到of_platform_bus_type总线上,由于此时of_platform_bus_type总线上还没有驱动,所以此时不进行匹配
 
int of_platform_populate(struct device_node *root,const struct of_device_id *matches,const struct of_dev_auxdata *lookup,struct device *parent)
//遍历of_allnodes中的所有节点，生成并初始化所以节点的platform_device结构

struct platform_device* of_find_device_by_node(struct device_node *np)
//根据device_node查找返回该设备对应的platform_device结构
```
