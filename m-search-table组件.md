# M-search-table 组件使用

## Demo

```vue
<template>
  <Card>
    <MSearchTable :render-data="renderData" />
  </Card>
</template>

<script>
import { defineComponent, getCurrentInstance } from 'vue'
import * as dayjs from 'dayjs'
 
export default defineComponent({
  name: 'SaleRecords', 
  components: {
    MSearchTable:()=>import('@/components/m-search-table/index.vue'),
  },
  setup() {
    const { proxy } = getCurrentInstance()
    const importFoo = ()=>{
      console.log('导入') 
    }
    const exportFoo = ()=>{
      console.log('导出')
    }
    const del = ({ row })=>{
      console.log(row, 'del')
    }
    const look = ({ row })=>{
      console.log(row, 'look')
    }
    const renderData = {
      table:[
        { type:'checkbox', width:50 },
        { title: '档位1', key: 'key1', formatter:({ row, h })=>{
          return h('span', { style:'color:red' }, '123123')
        } }, // 可以使用h函数渲染 return值也可以为字符串
        { title: '档位2',
          key: 'productId',
          type:'button', 
          slot:'productId',
          callback:({ row, selected })=>{
            console.log(row, selected) // 行数据 和 多选框数据
          },
        },
        { title: '档位3', key: 'key3' },
        { title: '档位4', key: 'key4' },
      ],
      api:{
        exec: proxy.$serverApi.getPhysicalAssessmentProjectList, // 请求函数
        subParams:{
          aaa:'123123', //游离参数
        },
      },
      lineOperation:{ // 编辑按钮 可以为数组也可以为对象 
        lineBtnMax:2, // 控制最多显示按钮 超出显示更多
        btnList: [
          { 
            label:'按钮1',
            type:'error',
            callback: del, 
            permission:['ManagePointSchemes'],
            disabled:({ row })=> row.age > 5, // disabled 属性可以为函数动态返回 也可以 为布尔值
            hide:({ row })=> row.age <= 5, // hide 属性 控制按钮显示隐藏 可以为函数动态返回 也可以 为布尔值
          },
          { 
            label:'按钮2',
            callback: look, 
            permission:['ManagePointSchemes'],
            disabled:({ row })=>{
              return row.isDis
            },
            hide:({ row })=>{
              return row.age > 8
            },
          },
          { 
            label:'按钮3',
            callback: look, 
          },
        ],
      },
      operation:{ // 表头按钮
        leftMax:4, // 左侧最多显示按钮个数 其余显示更多
        btnList:[
          { label: '导入体测项目', 
            callback: importFoo,
            type:'primary',
            permission:['ManagePhysicalTestingProjects'],
            float:'left', // 左侧按钮
          },
          { label: '导出体测项目',
            callback: exportFoo, 
            type:'error',
            permission: ['ExportVolumeProject'], 
            float:'right', // 右侧按钮
          },
        ],
      },
      search:[
        { name: '课程', key: 'courseId', type: 'course' }, 
        { name: '课程11', key: 'courseId22', type: 'Input',  canBeRest:false },
        { name: '老师', key: 'teacherId', type: 'allUser' },
        { name: '班级', key: 'classId', type: 'class' },
        { name: '课节状态', key: 'lessonStatus', type: 'enums', enumsValue: 'lessonStatus' },
		{ name: '端口', 
          key: 'ccc',
          type: 'select', 
          nameKey:'sourceName',
          valueKey:'id', 
          apiCallback:async ({ searchs })=>{
            console.log(searchs, 'searchs')
            const res = await proxy.$serverApi.getStudentSource()
            return res
          },
        },
        { name: '时间', 
          key: 'time', 
          type: 'daterange', 
          clearable: false,
          canBeRest: true, 
          needMore: true, 
          defaultValue:[dayjs().format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')],
          callback:(param)=>{ // 新增callback函数 处理数据
            return {
              startTime:param[0],
              endTime:param[1],
            }
          },
        },
      ],
      vxeTableProps:{
        custom:true, // vxe表格配置属性 传入
      },
    }
    return {
      renderData,
    }
  },
})
</script>

```

## Props

**renderData**    主体对象

|      key      |      type       | must  | default |       explain        |
| :-----------: | :-------------: | :---: | :-----: | :------------------: |
|     table     |      Array      | true  |         |   table列渲染数据    |
|      api      |     Object      | true  |         |  发送请求,接口配置   |
| lineOperation | Object \| Array | false |   []    |     编辑操作按钮     |
|   opration    | Object \| Array | false |   []    |     表头操作按钮     |
|    search     |      Array      | false |         |     搜索渲染对象     |
| vxeTableProps |     Object      | false |         | vxe原生对象props入口 |
|   hasPaging   |     Boolean     | false |  true   |     是否开启分页     |

**table**

|    key    |   type   | must  | default |                         explain                         |
| :-------: | :------: | :---: | :-----: | :-----------------------------------------------------: |
|   title   |  String  | true  |         |                        渲染名称                         |
|   type    |  String  | false |         |    值为 "button" , "checkbox" 特殊处理,后续继续增加     |
|    key    |  String  | true  |         |                    渲染字段 尽量唯一                    |
|   slot    |  String  | false |         |  插槽名称  type值为"button"时 , slot需与key值保持一致   |
| callback  | Function | false |         |               type值为"button"时, 可调用                |
| formatter | Function | false |         | 入参为{row,h} ,可以返回一个CreatedElement对象或者String |

**api**

|     key     |   type   | must  | default |                  explain                   |
| :---------: | :------: | :---: | :-----: | :----------------------------------------: |
|    exec     | Function | true  |         |               项目中请求函数               |
|  subParams  |  Object  | false |         |         每次请求会添加上的游离参数         |
| afterSearch | Function | false |         | 入参为(params:搜索参数 , res:接口返回参数) |

**lineOperation**   

width属性自动计算 无需手动赋值

|     key      |        type         | must  | default |                           explain                            |
| :----------: | :-----------------: | :---: | :-----: | :----------------------------------------------------------: |
|  lineBtnMax  |       Number        | false |    3    | 编辑按钮最多可显示数量<br /> 如果该选项有值 lineOperation为Object反之则为Array<br /> 整体字段变更 lineOperation:{ lineBtnMax:3,btnList:[item] } |
|    label     |       String        | true  |         |                           按钮名称                           |
|     type     |       String        | false |         |               暂时只提供'error'渲染按钮为红色                |
|  permission  |   Array \| String   | false |         |                           权限控制                           |
|   callback   |      Function       | true  |         |          点击事件回调函数 入参有 { row , selected }          |
|   disabled   | Boolean \| Function | false |  false  |           disabled属性控制,可以通过row数据动态返回           |
|     hide     | Boolean \| Function | false |  false  |              控制操作按钮显示隐藏,可以动态控制               |
| lineBtnWidth |       Number        | false |         |       编辑宽度 如要填写规则与lineBtnMax一致 需改变结构       |

**operation**

基础结构同 lineOperation

|   key   |       type        | must  | default |                           explain                            |
| :-----: | :---------------: | :---: | :-----: | :----------------------------------------------------------: |
| leftMax |      Number       | false |    4    | 左侧按钮最多显示数量<br /> 如果该字段有值  operation为Object,反之为Array<br /> 整体字段变更 operation:{ leftMax:3,btnList:[item] } |
|  float  | 'left' \| 'right' | false | 'left'  |              表头按钮渲染左边还是右边 依次排列               |

**search**

数据结构与 M-Search-Plus 一致  下面为新增Props

|      key      |           type           | must  |                  default                  |                           explain                            |
| :-----------: | :----------------------: | :---: | :---------------------------------------: | :----------------------------------------------------------: |
|   callback    |         Function         | false |                                           | 入参为 searchFrom[item.key] 该字段的值可以操作数据返回 <br /> 返回值需为Object |
|  apiCallback  | AsyncFunction \| Promise | false |                                           | 内部可以调用接口将接口返回的数据 return 或 resolve 出去<br />返回值可以为数组 或者 对象中带有data的数组如 { data:[item] }<br /> **如果需要用到此API需要 将type设置成"select",并设置nameKey和valueKey的值** |
|    nameKey    |          String          | false |                  'name'                   |        只在有apiCallback参数生效  决定下拉框的label值        |
|   valueKey    |          String          | false |                  'value'                  |        只在有apiCallback参数生效  决定下拉框的value值        |
| popoverConfig |          Object          | false | {content:String ,icon:'el-icon-question'} |         需传入contant为Tooltip渲染内容,icon可自定义          |





# m-modal 组件使用

## Demo

```vue
<template>
  <div>
    <Button @click="onTouchme">
      别摸我
    </Button>
  </div>
</template>

<script>
import touch from './touch.vue'
import { MModal } from '@/components/m-modal/index'

export default {
  methods:{
    onTouchme() {
      MModal({
        size:'mini',
        title:'被摸了',
        render:touch,
        defaultProps:{ toucher:'在座的每个人' },
        onSure:()=>{console.log('被摸完了 确认的回调')},
        onCancel:()=>{console.log('不摸了 取消的回调')},
      })
    },
  },
}
</script>
```

touch.vue

```vue
<template>
  <div>
    <h1>摸我的人 :{{ toucher }}</h1>
  </div>
</template>

<script>
export default {
  props:{
    toucher:{ type:String },
  },
  methods:{
    sure() {
      alert(this.toucher)
    },
  },
}
</script>
```

## Props

调用弹窗时 传入的对象结构

| key          | type                                        | must  | default                                                      | explain                        |
| ------------ | ------------------------------------------- | ----- | ------------------------------------------------------------ | ------------------------------ |
| size         | 'mini' \| 'small' \| 'medium'               | false | 'mini'                                                       | 内置弹窗3种宽度                |
| width        | String \| Number                            | false |                                                              | 控制弹窗宽度                   |
| title        | String                                      | true  |                                                              | 弹窗title                      |
| render       | VueComponent \| (h)=>CreatElement \| String | true  |                                                              | 渲染弹窗内容                   |
| btns         | Array:[btnItem]                             | false | [{label:'取消',value:'cancel},<br /> {label:'确认',value:'sure'}] | 弹窗footer                     |
| onSure       | Function                                    | false |                                                              | 点击确认回调                   |
| onCancel     | Function                                    | false |                                                              | 点击取消回调                   |
| defaultProps | Object                                      | false |                                                              | 传入render组件内部 props的属性 |

btns 用做 footer 内自定义按钮 改字段会影响后续回调事件字段 

因为 btns[n].value 默认为 'sure'           所以 回调函数字段为 'onSure'

如果 btns[n].value 为 'ok'                      那么 回调函数字段为 'onOk' 

同时 如果render 如果传入 是VueComponent时  在组组件内部也会触发  与 btn[n].value 同名方法

| function      | parameter | return | explain                                                      |
| ------------- | --------- | ------ | ------------------------------------------------------------ |
| ${btnValue}   |           | any    | 该函数为渲染组件内与 btnItem.value对应的回调<br /> 如果改函数为异步 则自动渲染loading,<br /> return 值为 true 时 则无法关闭弹窗 |
| on${btnValue} |           | any    | 该函数为事件穿透函数 <br /> render 如果传入 是VueComponent时 会先触发组件内部${btnValue}函数<br /> render 是其他值时  则会直接触发改函数 |
| onClose       |           |        | 点击x关闭回调                                                |
| onOpen        |           |        | 弹窗开启回调                                                 |

## close手动关闭 (不常用)

在某些特殊场景下 需要自己主动触发弹窗 此时可以使用 MModal.close() 来关闭所有弹窗 MModal 原型可用方法如下

| function | parameter | return | explain      |
| -------- | --------- | ------ | ------------ |
| close    |           |        | 关闭所有弹窗 |
| back     |           |        | 关闭当前弹窗 |

















