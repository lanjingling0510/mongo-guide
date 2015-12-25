# mongoose API

## TOC
- [查询接口](#find0)
- [修改接口](#update0)


已User和Beacon数据模型为例
``` script
const UserSchema = mongoose.Schema({
  username: {
    type: String,
    unique: true
  },
  passhash: String,
  nickname: String,
  fans: [{
    type: Schema.Types.ObjectId,
    ref: 'User'
  }]
});

const Beacon = mongoose.Schema({
  location: {
    type: [Number, Number],
    index: '2d'
  }
});

```


## <a id="find0"></a>查询接口
方法有：
1. find
2. findOne
3. findById
4. aggregate


### 过滤(match)

``` script

/* 正则表达式过滤 */
  const match = {
    'nickname': {
      $regex: 'xxx'
    }
  };

/* 数组过滤 */
  const match = {
    'fans': {
      $in: [new ObjectId(xxxxxxx)]
    }
  }


  User
  .aggregate()
  .match(match);

```

### 选择属性(project, select)

可以指定需要的的属性
还有更多的功能,例如:( 可以返回数组属性的长度， 返回数组属性的一部分)

``` script

  <!-- aggregate方法中 -->
    User.aggregate()
    .match({...})
    .project({
      nickname: 1,
      fansNumber: {
        $size: '$fans'
      }
    });

  <!-- find / findOne / findById方法中 -->
    const project = {
      'fans': $slice: [limit * page, limit * page + limit]
    }

    User.findOne({_id: new ObjectId(_id)},  project);
    // 或
    User.findOne({_id: new ObjectId(_id))
        .select('_id nickname fans');

```

### 将引用的数据集合填充到属性中(populate)

``` script
  User
  .findById(_id, project)
  .populate('fans');

  //  引用user集合返回nickname字段
  User
  .findById(_id, project)
  .populate('fans', 'nickname');
```

### 平面坐标区域过滤(goeWithin)
注意：纬度越上面越大，和canvas坐标系相反

``` script
    const sw = [parseFloat(54.5345), parseFloat(54.3434)];
    const ne = [parseFloat(34.4343), parseFoat(43.3464)];

    cosnt match = {
      'location': {
        $geoWithin: {
          $box: [
            [sw[0], sw[1]],
            [ne[0], ne[1]]
          ]
        }
      }
    };

    Beacon.find(match);

```

### 平面坐标区域距离排序(near)

``` script
  <!-- aggregate方法的实现方式 -->
  Share.aggregate([
     {
         $geoNear: {
             near: [543, 5435],
             distanceField: 'distance',
             maxDistance: (ne[0] - sw[0]),
             query: match,
             num: query.limit,
             uniqueDocs: true
         }
     }
  ]).exec();


  <!-- find方法的实现方式 -->
    const match = {
      location: {
        $near: [543, 5435], // 中心点
        $maxDistance: ne[1] - sw[1]
      }
    };

    Beacon.find(match);
```


## <a id="update0"></a>修改接口

### 修改数据集合属性
``` script
  <!-- 插入数据到数组属性的指定位置 -->
    User.findByIdAndUpdate(_id, {
      $push: {
        fans: {
          $each: [_userId],
          $position: 0
        }
      }
    });

  <!-- 删除数据从数组属性的指定位置 -->
    User.findByIdAndUpdate(_id, {
      $pull: {
        fans: _userId
      }
    })    
```

