


```javascript
{
	project_id : "str",
	user_id : "str",
	folder_id : 0,
	total : 100,
	pendig : 100,
	success : 0,
	fail : 0,
	
	files:[
		{
			dirHandle : 폴더핸들객체,
			file_list:[]
		},
		{
			dirHandle : 폴더핸들객체,
			file_list:[]
		},
	],
	file_idx : {
		"/path/file.jpg" : 0,
		...
		"/path/file100.jpg" : 99,
	}
}
```