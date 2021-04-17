#mongoose笔记
摘要:[http://www.jb51.net/article/80296.htm](http://www.jb51.net/article/80296.htm "摘要")

##安装&初始化

*注意：uri的格式类似：“mongodb://user:pass@localhost:port/database”。*

	var mongoose=require('mongoose');//引入模块
	mongoose.Promise = global.Promise;//屏蔽报错
	var db = mongoose.connect('mongodb://192.168.1.244:27017/test2');//初始化连接数据库,如果数据库不存在就创建一个
	//返回连接成功与否
	db.connection.on('error',function(error){
	    console.log('数据库连接失败'+error);
	});
	db.connection.on('open',function(){
	    console.log('数据库连接成功');
	});
##定义Schema

*注意：通俗讲就是字段的属性,未定义字段的不能插入数据库，_id为主键，如果不定义默认自动生成*

	var usersSchema = new mongoose.Schema({
		name:{type:String},//或者 name:String
		age:{type:Number,default:0}
		})
##定义Model
*注意：调用mongoose.model来编译Schema得到Model对象。定义Schema时指定的collection名字与mongoose.model的第一参数要保持一致。*
    
    //db.model('参数1：集合名称','参数2 :Schema实例')
	var usersModel = db.model('users',usersSchema);
#操作数据库 增删改查

*注意：以下的步骤都是基于以上的操作已完成*

##添加记录 基于Entity操作
*注意：Model和Entity都有能影响数据库的操作，但Model比Entity更具操作性*

    var doc={name:'meng',age:18};
	var usersEntity = new usersModel(doc);
	usersEntity.save(function(err,doc){
	    if(err){
	        console.log(err);
	    }else{
	        console.log(doc);
	    }
        //关闭数据库
        mongoose.disconnect();
	})
	
##添加记录 基于Model操作
    var doc={name:'meng',age:18};
    usersModel.create(doc,function(err){
        if(err){
            console.log(err);
        }else{
            console.log('save ok');
        }
        //关闭数据库
        mongoose.disconnect();
    });

##修改记录

**options参数详解**

upsert:如果不存在记录，是否插入新记录,true为插入，默认false不插入
multi:是否更新找到的所有记录,默认false只更新一条,true为全部更新


    //usersModel.update(查询条件,更新对象,其他选项,回调函数);usersModel.update(conditions, update, options, callback);
    var conditions={name:'meng'};
    var update={$set:{age:400}};
    // var options={};
    var options={multi:true};
    usersModel.update(conditions,update,options,function(err){
        if(err){
            console.log(err);
        }else{
            console.log('update ok!');
        }
    });

##删除记录
    //Model.remove(conditions, callback);
    var conditions={name:'meng'};
    usersModel.remove(conditions,function(err){
        if(err){
            console.log(err);
        }else{
            console.log('delete ok');
        }
    });
##查询记录

    Model.find(query, fields, options, callback)
    // fields 和 options 都是可选参数
    Model.find({ 'csser.com': 5 }, function (err, docs) { // docs 是查询的结果数组 });
    Model.find({}, ['first', 'last'], function (err, docs) {
      // docs 此时只包含文档的部分键值
    })
    Model.findOne({ age: 5}, function (err, doc){
      // doc 是单个文档
    });
    Model.findById(obj._id, function (err, doc){
      // doc 是单个文档
    });
    Model.count(conditions, callback);
    复杂查询
    Model
    .where('age').gte(25)
    .where('tags').in(['movie', 'music', 'art'])
    .select('name', 'age', 'tags')
    .skip(20)
    .limit(10)
    .asc('age')
    .slaveOk()
    .hint({ age: 1, name: 1 })
    .run(callback);
    
    //现在要分页查询，每页3条，查询第2页
	//skip 跳过的条数 limit 限制返回的条数 sort排序 1升序 -1 降序
	PersonModel.find({},{_id:0,name:1},{limit:3,skip:3,sort:{age:1,name:-1}},function(err,docs){
	    console.log(docs);
	});








**更新修改器**

$inc 自增
*Model.update({‘age’:22}, {’$inc’:{‘age’:1} }  ); 执行后: age=23*

$set 设置
*Model.update({‘age’:22}, {’$set’:{‘age’:‘haha’} }  ); 执行后: age=‘haha’*

$unset 删除
*Model.update({‘age’:22}, {’$unset’:{‘age’:‘haha’} }  ); 执行后: age键不存在*

**数组修改器**

1. Model.update({‘age’:22}, {’$push’:{‘array’:10} } ); 执行后: 增加一个 array 键,类型为数组, 有一个成员 10*
2. Model.update({‘age’:22}, {’$addToSet’:{‘array’:10} } ); 执行后:不存在就添加， array中有10所以不会添加*
3. Model.update({‘age’:22}, {’$push’:{‘array’:{’$each’: [1,2,3,4,5]}} } ); 执行后: array : [10,1,2,3,4,5]*
4. Model.update({‘age’:22}, {’$pop’:{‘array’:1} } ); 执行后: array : [10,1,2,3,4] tips: 将1改成-1可以删除数组首部元素*
5. Model.update({‘age’:22}, {’$pull’:{‘array’:10} } ); 执行后: array : [1,2,3,4] 匹配到array中的10后将其删除*

**条件查询**

 - “$lt” 小于
 - “$lte” 小于等于
 - “$gt” 大于
 - “$gte” 大于等于
 - “$ne” 不等于

*Model.find({“age”:{ “$gte”:18 , “$lte”:30 } } ); 查询 age 大于等于18并小于等于30的文档*

**或查询 OR:**

- ‘$in’ 一个键对应多个值
- ‘$nin’ 同上取反, 一个键不对应指定值
- “$or” 多个条件匹配, 可以嵌套 $in 使用
- “$not”     同上取反, 查询与特定模式不匹配的文档

*Model.find({“age”:{ “$in”:[20,21,22.‘haha’]} } ); 查询 age等于20或21或21或’haha’的文档*
*Model.find({"$or" :  [ {‘age’:18} , {‘name’:‘xueyou’} ] }); 查询 age等于18 或 name等于’xueyou’ 的文档*

**类型查询**

Model.find(“age” :  { “$in” : [null] , “exists” : true  } ); 查询 age值为null的文档

**查询数组**

1. Model.find({“array”:10} ); 查询 array(数组类型)键中有10的文档, array : [1,2,3,4,5,10] 会匹配到*

2. Model.find({“array[5]”:10} ); 查询 array(数组类型)键中下标5对应的值是10, array : [1,2,3,4,5,10] 会匹配到

3. Model.find({“array”:[5,10]} ); 查询 匹配array数组中 既有5又有10的文档

4. Model.find({“array”:{"$size" : 3} } ); 查询 匹配array数组长度为3 的文档

5. Model.find({“array”:{"$skice" : 10} } ); 查询 匹配array数组的前10个元素

6. Model.find({“array”:{"$skice" : [5,10] } } ); 查询 匹配array数组的第5个到第10个元素

