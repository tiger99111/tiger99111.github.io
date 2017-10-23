---
layout: post
title: linux��devicetree�г��õ�of����
date: 2017-10-23 
tags: Ƕ��ʽlinux��������    
---


###linux��devicetree�г��õ�of����

```
//��device_node�л�ȡ��Ϣ��
int of_property_read_u8_array(const struct device_node *np, const char *propname,u8 *out_values, size_t sz);
int of_property_read_u16_array(const struct device_node *np, const char *propname,u16 *out_values, size_t sz);
int of_property_read_u32_array(const struct device_node *np, const char *propname,u32 *out_values, size_t sz);
//���豸���np�ж�ȡ������Ϊpropname������Ϊ8��16��32��λ�������������ֵ��������out_values��szָ����Ҫ��ȡ�ĸ�����

static inline int of_property_read_u8(const struct device_node *np,const char *propname,u8 *out_value) 
static inline int of_property_read_u16(const struct device_node *np,const char *propname,u8 *out_value) 
static inline int of_property_read_u32(const struct device_node *np,const char *propname,u8 *out_value) 
//���豸���np�ж�ȡ������Ϊpropname������Ϊ8��16��32λ������ֵ��������out_values��ʵ����������õľ���szΪ1��XXX_array������
 
int of_property_read_u32_index(const struct device_node *np,const char*propname,u32 index, u32 *out_value)
//���豸���np�ж�ȡ������Ϊpropname������ֵ�е�index��u32��ֵ��out_value
 
int of_property_read_u64(conststruct device_node *np, const char *propname,u64 *out_value)
//���豸���np�ж�ȡ������Ϊpropname������Ϊ64λ������ֵ��������out_values
 
int of_property_read_string(struct device_node *np, const char *propname,const char**out_string)
//���豸���np�ж�ȡ������Ϊpropname���ַ���������ֵ
 
int of_property_read_string_index(struct device_node *np, const char *propname,intindex, const char **output)
//���豸���np�ж�ȡ������Ϊpropname���ַ���������ֵ�����еĵ�index���ַ���
 
int of_property_count_strings(struct device_node *np, const char *propname)
//���豸���np�ж�ȡ������Ϊpropname���ַ���������ֵ�ĸ���
 
unsigned int irq_of_parse_and_map(struct device_node *dev, int index)
//���豸�ڵ�dev�ж�ȡ��index��irq��
 
int of_irq_to_resource(struct device_node *dev, int index, struct resource *r)
//���豸�ڵ�dev�ж�ȡ��index��irq�ţ������һ��irq��Դ�ṹ��
 
int of_irq_count(struct device_node *dev)
//��ȡ�豸�ڵ�dev��irq����

static inline bool of_property_read_bool(const struct device_node *np,const char *propname);
//����豸���np����propname���ԣ��򷵻�true�����򷵻�false��һ�����ڼ��������Ƿ���ڡ�
 
struct property* of_find_property(const struct device_node *np,const char *name,int *lenp)
//����name��������ָ�����豸���np�в���ƥ���property�����������property
 
const void * of_get_property(const struct device_node *np, const char *name,int *lenp)
//����name��������ָ�����豸���np�в���ƥ���property�����������property������ֵ

struct device_node* of_get_parent(const struct device_node *node)
//���node�ڵ�ĸ��ڵ��device node

int of_device_is_compatible(const struct device_node *device,const char *compat);
//�ж��豸���device��compatible�����Ƿ����compatָ�����ַ���

//��of_allnodes�в�����Ϣ��
struct device_node* of_find_node_by_path(const char *path)
//����·����������ȫ������of_allnodes�У�����ƥ���device_node

struct device_node* of_find_node_by_name(struct device_node *from,const char *name)
//�����name��ȫ������of_allnodes�в���ƥ���device_node,��from=NULL��ʾ��ͷ��ʼ����

struct device_node* of_find_node_by_type(struct device_node *from,const char *type)
//�����豸������ȫ������of_allnodes�в���ƥ���device_node

struct device_node * of_find_compatible_node(struct device_node *from, const char*type, const char��*compatible);
//����compatible������ֵ��ȫ������of_allnodes�в���ƥ���device_node�����������£�from��typeΪNULL��
 
struct device_node* of_find_node_with_property(struct device_node *from,const char *prop_name)
//���ݽڵ����Ե�name��ȫ������of_allnodes�в���ƥ���device_node
 
struct device_node* of_find_node_by_phandle(phandle handle)
//����phandle��ȫ������of_allnodes�в���ƥ���device_node
 
//�ӣ�
void __iomem* of_iomap(struct device_node *node, int index);
//ͨ���豸���ֱ�ӽ����豸�ڴ������ ioremap()��index���ڴ�ε����������豸����reg�����ж�Σ���ͨ��index��ʾҪioremap������һ�Σ�ֻ��1�ε������indexΪ0
 
unsigned long __init of_get_flat_dt_root(void)
//����������dtb�еĸ��ڵ㣬���񷵻صĶ���0

int of_alias_get_id(struct device_node *np, const char *stem)
//��ȡ�ڵ�np��Ӧ��aliasid��
 
struct device_node* of_node_get(struct device_node *node)
void of_node_put(struct device_node *node)
//device node��������/����

const struct of_device_id* of_match_node(const struct of_device_id *matches,const struct device_node*node)
//��matches������of_device_id�ṹ��name��type��device node��compatible��typeƥ�䣬����ƥ�����ߵ�of_device_id�ṹ

platform_device��resource��أ�
int of_address_to_resource(struct device_node *dev, int index,struct resource *r)
//�����豸�ڵ�dev��reg����ֵ�������Դ�ṹ��r��Index����ָ����ʹ��reg�����еڼ�������ֵ��һ������Ϊ0����ʾ��һ����

struct platform_device* of_device_alloc(struct device_node *np,const char *bus_id,struct device *parent)
//����device node��bus_id�Լ����ڵ㴴�����豸��platform_device�ṹ��ͬʱ���ʼ������resource��Ա��
 
int of_platform_bus_probe(struct device_node *root,const struct of_device_id *matches,struct device *parent)
//����of_allnodes�еĽڵ�ҽӵ�of_platform_bus_type������,���ڴ�ʱof_platform_bus_type�����ϻ�û������,���Դ�ʱ������ƥ��
 
int of_platform_populate(struct device_node *root,const struct of_device_id *matches,const struct of_dev_auxdata *lookup,struct device *parent)
//����of_allnodes�е����нڵ㣬���ɲ���ʼ�����Խڵ��platform_device�ṹ

struct platform_device* of_find_device_by_node(struct device_node *np)
//����device_node���ҷ��ظ��豸��Ӧ��platform_device�ṹ
```
**����ת��**��[http://www.myexception.cn/linux-unix/1910031.html](http://www.myexception.cn/linux-unix/1910031.html)

