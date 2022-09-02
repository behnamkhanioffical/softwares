| 键                            | 类型    | 描述                                                                                                              |
| ---------------------------- | ----- | --------------------------------------------------------------------------------------------------------------- |
| `action`                     | `字符串` | 执行的操作内容. Can be one of `created`, `closed`, `opened` (a closed milestone is re-opened), `edited`, or `deleted`. |
| `里程碑`                        | `对象`  | 里程碑本身。                                                                                                          |
| `changes`                    | `对象`  | 对里程碑的更改，如果操作为 `edited`。                                                                                         |
| `changes[description][from]` | `字符串` | 说明的先前版本（如果操作为 `edited`）。                                                                                        |
| `changes[due_on][from]`      | `字符串` | 到期日期的先前版本，如果操作为 `edited`。                                                                                       |
| `changes[title][from]`       | `字符串` | 标题的先前版本，如果操作为 `edited`。                                                                                         |