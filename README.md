kdb
====

kdb is a package to wrap Go's [database/sql](http://golang.org/pkg/database/sql)

## Version

Current release: version 0.2 (2013-07-31)

## Document

[godoc](http://godoc.org/github.com/sdming/kdb)  
Sorry for bad english, if you want to improve documents, please contact me.  

## Features  

* Lightweight, flexible and fast  
* ORM  
* Sql template  
* Store procedure 
  
## Requirements

Go 1.1+  

## Example 


	`

		func basic() {
			db := kdb.NewDB("demo")
			defer db.Close()

			var query string

			query = "select * from ttable where cint > ?"
			fmt.Println("\nQuery:", query)
			printRows(db.Query(query, 1))

			query = "update ttable set cdatetime=NOW() where cint > ?"
			fmt.Println("\nExec", query)
			printResult(db.Exec(query, 1))

			query = "select * from ttable where cint > {cint}"
			fmt.Println("\nQueryText", query)
			printRows(db.QueryText(query, kdb.Map(data)))

			query = "update ttable set cdatetime=NOW() where cint > {cint}"
			fmt.Println("\nExecText", query)
			printResult(db.ExecText(query, kdb.Map(data)))
		}

	`	

## Template

`
func text() {
	db := kdb.NewDB("demo")
	defer db.Close()

	query := "select * from ttable where cint > {cint}"
	text := kdb.NewText(query).Set("cint", 1)
	fmt.Println("\nText query", query)
	printRows(db.QueryExp(text))

	query = "update ttable set cdatetime=NOW() where cint > {cint}"
	text = kdb.NewText(query).Set("cint", 42)
	fmt.Println("\nText exec", query)
	printResult(db.ExecExp(text))

}

`

## Store Procedure  

`
func procedure() {
	db := kdb.NewDB("demo")
	fmt.Println("\nQueryFunc", "sp_query")
	printRows(db.QueryFunc("sp_query", kdb.Map(data)))
	db.Close()

	db = kdb.NewDB("demo")
	fmt.Println("\nExecFunc", "sp_exec")
	printResult(db.ExecFunc("sp_exec", kdb.Map(data)))
	db.Close()

	db = kdb.NewDB("demo")
	fmt.Println("\nProcedure", "sp_exec")
	sp := kdb.NewProcedure("sp_exec").
		Set("cint", 42)
	printResult(db.ExecExp(sp))
	db.Close()
}

`
## Select

`
func selectTable() {
	db := kdb.NewDB("demo")
	defer db.Close()

	fmt.Println("\nSelectAll")
	printRows(db.SelectAll("ttable",
		"cint", kdb.GreaterThan, 1,
		"cfloat", kdb.LessThan, 123456,
		"cdatetime", "<>", time.Now(),
	))

	fmt.Println("\nSelectCount")
	fmt.Println(db.SelectCount("ttable"))

	fmt.Println("\nSelectExists")
	fmt.Println(db.SelectExists("ttable", "cint", kdb.GreaterThan, 12345))

	fmt.Println("\nQuery ttable")
	q := kdb.NewQuery("ttable", "")
	q.Select.
		Column("cbool", "cint").
		ColumnAs("cstring", "astring").
		Exp(kdb.Sql("cfloat-1"), "afloat").
		Avg("cnumeric", "aavg").
		Count("cnumeric", "acount").
		Max("cnumeric", "amax").
		Min("cnumeric", "amin").
		Sum("cnumeric", "asum")

	q.Where.
		OpenParentheses().
		IsNull("cbytes").
		Or().
		IsNotNull("cbytes").
		CloseParentheses()

	q.UseGroupBy().
		Column("cbool", "cint").
		By(kdb.Sql("cfloat-1"))

	q.UseHaving().
		Count(kdb.GreaterThan, "cnumeric", 0).
		LessThan("cint", 12345)

	q.UseOrderBy().Asc("cbool", "cint").Desc("cstring")

	printRows(db.QueryExp(q))

}

`
## Update 

`
func updateTable() {
	db := kdb.NewDB("demo")
	defer db.Close()

	fmt.Println("\nUpdateColumn")
	fmt.Println(db.UpdateColumn("ttable", "cstring", "cstring_update", "cint", kdb.Equals, 42))

	fmt.Println("\nUpdate")
	fmt.Println(db.Update("ttable", kdb.Map(data), "cint", kdb.Equals, 42))

	fmt.Println("\nUpdate ttable")
	u := kdb.NewUpdate("ttable")
	u.Set("cstring", "cstring new").
		Set("cfloat", 6.28)
	u.Where.Equals("cint", 42)
	u.Limit(1000)

	printResult(db.ExecExp(u))

}

`

## Delete

`
func deleteTable() {
	db := kdb.NewDB("demo")
	defer db.Close()

	fmt.Println("\nDelete ttable by column")
	fmt.Println(db.DeleteByCol("ttable", "cint", 101))

	fmt.Println("\nDelete ttable by conditions")
	fmt.Println(db.Delete("ttable",
		"cint", kdb.Equals, 101,
		"cfloat", kdb.NotEquals, 3.14,
		"cstring", kdb.GreaterThan, "cstring",
		"cdatetime", "=", "2001-01-01",
	))

	fmt.Println("\nDelete ttable")
	del := kdb.NewDelete("ttable")
	del.Where.
		LessThan("cint", 10000).
		GreaterThan("cint", 101).
		NotEquals("cint", 12345).
		NotIn("cint", [5]int{5, 6, 7, 8, 9})
	printResult(db.ExecExp(del))

}

`

# Insert 

`
func insertTable() {
	db := kdb.NewDB("demo")
	defer db.Close()

	d1 := map[string]interface{}{
		"cbool":     true,
		"cint":      123,
		"cfloat":    1.23,
		"cnumeric":  12.30,
		"cstring":   "a string",
		"cdate":     "2000-01-23",
		"cdatetime": time.Now(),
	}

	fmt.Println("\nInsert")
	fmt.Println(db.Insert("ttable", kdb.Map(d1)))

	fmt.Println("\nInsert ttable")
	insert := kdb.NewInsert("ttable")
	insert.Set("cbool", 0).
		Set("cint", 12345).
		Set("cfloat", 12.345).
		Set("cnumeric", 1234.50).
		Set("cstring", "string insert").
		Set("cdate", "1979-01-02")

	printResult(db.ExecExp(insert))

}
`

## Installation

go get github.com/sdming/kdb 

## Driver

*Mysql: github.com/Go-SQL-Driver/MySQL 
*PostgreSQL: github.com/bmizerany/pq  
*SQLite: github.com/mattn/go-sqlite3  
*Oracle: github.com/mattn/go-oci8  
*Odbc: https://code.google.com/p/odbc 

## TODO  

* ORM  
* Store procedure output parameter 
* mssql, postgresql, oracle, sqlite  

## License

MIT