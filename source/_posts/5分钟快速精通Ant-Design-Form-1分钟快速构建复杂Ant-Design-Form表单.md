---
title: 5分钟快速精通Ant Design Form 1分钟快速构建复杂Ant Design Form表单
date: 2019-07-28 20:05:56
tags: [Ant-Design-Form, Ant-Design]
category: [Ant-Design]
cover: 
---
## Ant Design Form
Antd 表单的核心无非是以下两点

 1. 表单创建（`Form.create`）在`this.props`写入`form`属性
 2. 表单与组件的双向绑定（`this.props.form.getFieldDecorator`）
 3. 表单的取值（`this.props.form.validateFields / this.props.form.validateFieldsAndScroll`）

#### 表单创建
`Form.create`这是一个高阶函数，传入的是react组件，返回一个新的react组件，在函数内部会对传入组件进行改造，添加上一定的方法用于进行一些秘密操作 ，这里不多做解释，有兴趣的同学可以上官网查看。

[我是飞机票（React-高阶组件），点我](https://react.docschina.org/docs/higher-order-components.html)
使用方法如下：

```
class CustomizedForm extends React.Component { ... }

// use wrappedComponentRef
const EnhancedForm =  Form.create()(CustomizedForm);
```
或者
```
@Form.create()
class CustomizedForm extends React.Component { ... }
```
#### 表单与组件的双向绑定
他的目的是将表单的组件的值与表单绑定。最后表单可以直接取到某某组件的值。

```
<!-- 表单数据绑定 -->
<Form.Item {...formItemLayout} label={'名称'}>
	{getFieldDecorator('searchName')(
		<Input placeholder={'请输入名称'} />
	)}
</Form.Item>
```
这个是一个非常简单的绑定，组件input的值都会由'searchName'这个属性收纳。
可以直接取值，当然也可以加入自己的校验规则，等等。这里不多做解释，这些都是附加的进阶功能，这里不多做描述。

[我是飞机票（表单进阶），点我！](https://ant.design/components/form-cn/)

#### 表单的取值

```
this.props.form.validateFields((err, values) => {
      if (!err) { // 这里也可以不要，是用于校验的。
        console.log('Received values of form: ', values);
      }
    });
```
这是一个非常简单的取值，当然可以定制的取值，或者定制的校验，比如，在获取验证码的时候，不需要校验密码是否输入或者符合你的规则，就可以只校验手机号，等等。这里不多做解释，这些都是附加的进阶功能，这里不多做描述。

[我是飞机票（表单进阶），点我！](https://ant.design/components/form-cn/)

基本上你掌握了这些就可以较为灵活的使用Ant Design Form了。

## 但是！划重点了！
如果有一些更加进阶的想法。请看！

[表单的进阶使用](https://blog.csdn.net/cuandeqin2083/article/details/89643390)

# 重中之重
如果你觉得自己构建表单非常麻烦，或者对表单的理解还是够透彻，想一步到位构件表单，推荐本人自用自创组件
ant-design-form

上面的文档非常详细，如果有问题随时可以给我提issues。如果喜欢的话麻烦点个start~

[我是ant-design-from的飞机票，点我点我！！](https://github.com/DerrickTel/ant-design-form)