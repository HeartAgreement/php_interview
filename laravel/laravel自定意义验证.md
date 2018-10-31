#验证速查
##一般验证有三种方式
1. 控制器中$this->validate(object $Request,array 验证规则，array 验证错误信息)
2. request验证器
3. Validator::make（）原型验证
##视图默认会有一个$errors的变量
```
count($errors) > 0判断是否有值 （对象）
$errors->first('input_name')获取一个错误信息（多个错误信息中获取第一个）
$errors->all（）获取全部错误信息
$errors->has('input_name')判断是否有错误信息
$errors->get('input_name')返回是一个数组（根据错误信息有可能是多维）
```
##验证器
```
php artisan make:request StoreBlogPost
//request验证器规则
```
```

/**
 *//获取适用于请求的验证规则。
 * @return array
 */
public function rules()
{ 
    return [
        'test' => 'required|unique:posts|max:255',
    ];
}
 
/**
 * 配置验证器实例。

 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{//验证后的钩子 
    $validator->after(function ($validator) {
    });
}
 
/**
 * 判断用户是否有权限做出此请求。
 * @return bool
 */
public function authorize()
{
}
 
/**
 * 获取已定义的验证规则的错误消息。
 * @return array
 */
public function messages()
{
}
```
##自定义验证规则
```
php artisan make:rule Uppercase //函数形式
```

```
App\Providers\AppServiceProvider
    /**
     * 引导任何应用服务。
     * @return void
     */
    public function boot()
    {
        //$validator->getData();获取所有数据内容   
        //                   '被验证的字段名.*'=>'验证调用的名字:参数1，参数2'   $validator是验证器对象                   
        Validator::extend('验证调用的名字', function (string 被验证的字段名$attribute, 被验证的字段值$value,array 参数1$parameters, object $validator) {
            return bool;
        });//闭包式的验证
        //                         对象@方法 方法结构(就跟闭包式的验证一样)               
        Validator::extend('foo', 'FooValidator@validate');
        //注册错误信息 没用过 感觉没必要
        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }
```



