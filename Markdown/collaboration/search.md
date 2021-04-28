<!--
 * @Author: jhliang
 * @Descripttion: In User Settings Edit
 * @LastEditTime: 2020-12-19 23:55:55
 * @Date: 2020-12-17 17:05:23
 * @LastEditors: jhliang
 * @FilePath: \MyBook\Markdown\collaboration\search.md
-->

# 输入参数

````json
{
  // 导出字段值列表，非必要
  "exportFieldCodes": [],
  // 输入查询参数
  "searchArgs": {
    "assignee": "", // 处理人，传用户id
    "reporter": "", // 负责人，传用户id
    
    "status": "", // 状态
    "typeCode": "", // 类型
    "createdBy":"", //创建者id，传用户id
    "priority": "", // 优先级数组，有可能是数组

    "storyPoints": "", // 故事点
    "remainingTime": "", //剩余时间
    "onlyStory": "",  // 仅story
    "updateStartDate": "", // 最后更新时间过滤，和updateEndDate组成一个区间
    "updateEndDate": "", // 和上面一个区间
    "createStartDate": "", // 创建时间过滤，和createEndDate组成一个区间
    "createEndDate": "", // 和上面一个区间
    "version": "", // 版本名称过滤
    "component": "", // 模块名称过滤
    "sprint": "", // 迭代名称过滤
    "issueNum": "", // 迭代编号
    "summary": "", // 标题过滤

    "parentIssueId": "0" // 查询可以关联的子工作项（只能关联没有父工作项的工作项，指代issueid = 0），也可以传入其他id，查出这些id的子工作项列表
  },
  // 过滤查询参数
  "advancedSearchArgs": {},
  // 关联查询参数
  "otherArgs": {},
  // 快速筛选id列表
  "quickFilterIds": [],
  // 处理人筛选id列表
  "assigneeFilterIds": [],
  // 仅需求
  "onlyStory": boolean,
  // 内容
  "content": "",
  // 内容列表
  "contents": [],
}
````
