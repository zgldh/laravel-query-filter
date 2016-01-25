# laravel-query-filter

An easier way to filter Eloquent and Models. Save you from Request::input and Where hell.

再也不用写一长串where语句了，再也不用查询语句贴来贴去了。

## 用法：

```PHP

    use zgldh\LaravelQueryFilter\AbstractFilter;
    
    //首先要定义一个过滤器， 继承自 AbstractFilter
    class UserFilter extends AbstractFilter     
    {
        //该函数必须被重写， 用于定义哪些字段需要被过滤
        public function columns()               
        {
            $columns = [
                // 参数名为 name， 数据库字段名为 username, 过滤规则为 “LIKE” (前后百分号)
                Column::build('name', 'username', Column::LIKE),        
                // 参数名为 age, 数据库字段名为 age， 过滤规则为 “等于”
                Column::build('age'),                                   
                // 参数名为 email, 数据库字段名为 email (其实无所谓), 过滤规则为回调函数 emailCallback()
                Column::build('email', null, [$this, 'emailCallback']), 
                // 参数名为 created_at, 数据库字段名为 created_at, 过滤规则为 "Between"
                Column::build('created_at', null, Column::BETWEEN),
                // 参数名为 status, 数据库字段名为 status, 规则为 “等于”，默认值为 'activate'
                Column::build('status', null, Column::EQUAL)
                    ->setDefaultValue('activate')                        
            ];
            return $columns;
        }
        
        /**
         * @param $builder          Eloquent Builder
         * @param Column $column    上面定义的这一列规则
         * @param $value            过滤的参数值
         * @return mixed            要返回一个Eloquent Builder
         */
        public function emailCallback($builder, Column $column, $value) 
        {
            //这里就跟平常写where一样。
            return $builder->where('email', 'LIKE', $value.'%');    
        }
    }
    
    //实例化过滤器
    $filter = new UserFilter();
    
    //应用过滤器，得到最终结果
    $users = $filter->filter(new User(), \Request::all())->get();
```

## 安装

``` composer require laravel-query-filter ```


## 过滤规则

### LIKE 
    
会生成 ```PHP $builder->where('key', 'like', '%'.$value.'%') ``` 的查询。 

### EQUAL 

会生成 ```PHP $builder->where('key', $value) ``` 的查询。 

### NOT_EQUAL 

会生成 ```PHP $builder->where('key', '<>', $value) ``` 的查询。 
    
### GREATER_THAN 

会生成 ```PHP $builder->where('key', '>', $value) ``` 的查询。 
    
### LESS_THAN 

会生成 ```PHP $builder->where('key', '<', $value) ``` 的查询。 
    
### BETWEEN 

要求传入参数为 ``` $value = ['start'=>123, 'end'=>456] ``` 的形式
会生成
    
    ```PHP
        
        $builder->where('key', '>=', $value['start']) 
                ->where('key', '<=', $value['end']) 
    
    ``` 
    
的查询。 
    
### CALLBACK 

会跳入 callback 函数， 执行其中的查询。 
    
    
## 技巧

有时从前端传来的过滤参数被包裹在一个数组里。 如：
```PHP

    [
        'filters'=>[
            'name'=>'Wang',
            'age'=>20
            'email'=>null
        ]
    ]
    
```

则我们可以为该过滤器添加命名空间来自动脱掉外层的 filters：


```PHP

    use zgldh\LaravelQueryFilter\AbstractFilter;
    class UserFilter extends AbstractFilter     
    {
        public $namespace = 'filters';     // 会自动脱去外层的 filters
    
        public function columns()               
        {
        ...
```

待续
