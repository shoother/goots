UpdateRow
=========
	
	// 说明：更新一行数据。
	//
	// ``table_name``是对应的表名。
	// ``condition``表示执行操作前做条件检查，满足条件才执行，是string的实例。
	// 目前只支持对行的存在性进行检查，检查条件包括：'IGNORE'，'EXPECT_EXIST'和'EXPECT_NOT_EXIST'。
	// ``primary_key``表示主键，类型为``otstype.OTSPrimaryKey``的实例。
	// ``update_of_attribute_columns``表示属性列，类型为``otstype.OTSUpdateOfAttribute``的实例，可以包含put和delete操作。其中put是
	// ``otstype.OTSColumnsToPut`` 表示属性列的写入；delete是``otstype.OTSColumnsToDelete``，表示要删除的属性列的列名，
	// 见示例。
	//
	// 返回：本次操作消耗的CapacityUnit。
	//       错误信息。
	//
	// ``update_row_response``为``otstype.OTSUpdateRowResponse``类的实例包含了：
	// ``Consumed``表示消耗的CapacityUnit，是``otstype.OTSCapacityUnit``类的实例。
	//
	// 示例：
	//
	// primary_key := &OTSPrimaryKey{
	// 	"gid": 1,
	// 	"uid": 101,
	// }
	// update_of_attribute_columns := &OTSUpdateOfAttribute{
	// 	OTSOperationType_PUT: OTSColumnsToPut{
	// 		"name":    "张三丰",
	// 		"address": "中国B地",
	// 	},
	//
	// 	OTSOperationType_DELETE: OTSColumnsToDelete{
	// 		"mobile", "age",
	// 	},
	// }
	// condition := OTSCondition_EXPECT_EXIST
	// update_row_response, ots_err := ots_client.UpdateRow("myTable", condition, primary_key, update_of_attribute_columns)
	//
	func (o *OTSClient) UpdateRow(table_name string, condition string, primary_key *OTSPrimaryKey, update_of_attribute_columns *OTSUpdateOfAttribute) (update_row_response *OTSUpdateRowResponse, err *OTSError)

Example
=======
[UpdateRow.go](https://github.com/shoother/goots/blob/master/example/8-UpdateRow.go)

	package main
	
	import (
		"fmt"
		"os"
	
		ots2 "github.com/shoother/goots"
		"github.com/shoother/goots/log"
		. "github.com/shoother/goots/otstype"
	)
	
	// modify it to yours
	const (
		ENDPOINT     = "http://127.0.0.1:8800"
		ACCESSID     = "OTSMultiUser177_accessid"
		ACCESSKEY    = "OTSMultiUser177_accesskey"
		INSTANCENAME = "TestInstance177"
	)
	
	func main() {
		// set running environment
		ots2.OTSDebugEnable = true
		ots2.OTSLoggerEnable = true
		log.OTSErrorPanicMode = true // 默认为开启，如果不喜欢panic则设置此为false
	
		fmt.Println("Test goots start ...")
	
		ots_client, err := ots2.New(ENDPOINT, ACCESSID, ACCESSKEY, INSTANCENAME)
		if err != nil {
			fmt.Println(err)
		}
	
		// update_row
		primary_key := &OTSPrimaryKey{
			"gid": 1,
			"uid": 101,
		}
		update_of_attribute_columns := &OTSUpdateOfAttribute{
			OTSOperationType_PUT: OTSColumnsToPut{
				"name":    "张三丰",
				"address": "中国B地",
			},
	
			OTSOperationType_DELETE: OTSColumnsToDelete{
				"mobile", "age",
			},
		}
		condition := OTSCondition_EXPECT_EXIST
		update_row_response, ots_err := ots_client.UpdateRow("myTable", condition, primary_key, update_of_attribute_columns)
		if ots_err != nil {
			fmt.Println(ots_err)
			os.Exit(1)
		}
		fmt.Println("成功插入数据，消耗的写CapacityUnit为:", update_row_response.GetWriteConsumed())
	
		// get_row
		primary_key = &OTSPrimaryKey{
			"gid": 1,
			"uid": 101,
		}
		columns_to_get := &OTSColumnsToGet{
			"name", "address", "age",
		}
		columns_to_get = nil // read all
		get_row_response, ots_err := ots_client.GetRow("myTable", primary_key, columns_to_get)
		if ots_err != nil {
			fmt.Println(ots_err)
			os.Exit(1)
		}
		fmt.Println("成功读取数据，消耗的读CapacityUnit为:", get_row_response.GetReadConsumed())
		if get_row_response.Row != nil {
			if attribute_columns := get_row_response.Row.GetAttributeColumns(); attribute_columns != nil {
				fmt.Println("name信息:", attribute_columns.Get("name"))
				fmt.Println("address信息:", attribute_columns.Get("address"))
				fmt.Println("age信息:", attribute_columns.Get("age"))
				fmt.Println("mobile信息:", attribute_columns.Get("mobile"))
			} else {
				fmt.Println("未查询到数据")
			}
		} else {
			fmt.Println("未查询到数据")
		}
	}