# Light_datatables
This library based on CodeIgniter3  that will help you use jQuery Datatables in server side mode.
## 如何部屬
### 文件
../application/libraries/Light_datatables.php
### Controllers

```php
$this->load->library('light_datatables');
```

## 快速開始
[使用說明](https://hackmd.io/s/BJNnBnndQ)
### 基本的Server Side Data Tables
在以下的程式中，我們以一個簡單的佈告欄作為範例，展示一個基本的 Data Tables 於 Server Side模式中的使用方法。
#### HTML and JavaScript
```javascript=0
<table id="noticeTable">
    <thead>
        <tr>
            <th>標題</th>
            <th>發布時間</th>
        </tr>
    </thead>
</table>
<script>
    $('#noticeTable').DataTable({
        "ajax":{
             url:base_url('notice/datatable'),
             type:'POST'
        }
    });
</script>
```
#### Database
```sql=0
CREATE TABLE `notice`  (
    `key` int(11) NOT NULL AUTO_INCREMENT,
    `title` varchar(255) NOT NULL,
    `content` text CHARACTER  NOT NULL,
    `time` datetime(0) NOT NULL DEFAULT '0000-00-00 00:00:00',
    PRIMARY KEY (`key`)
)
```
#### php
```php=0
public function datatable(){
    $order=array('title','time');
    $like=array('title','time');
    $output=array('title','time');
    $this->light_datatables->ci->db->from('notice');
    $this->light_datatables->ci->db->select('title, time');
    $this->light_datatables->set_querycolumn($order,$like);
    $this->light_datatables->order_by('time','DESC');
    $this->light_datatables->set_output($output);
    echo $this->light_datatables->get_datatable();
}
```
## functions


|  呼叫名稱 |   傳入值  |   說明  |   回傳值  |
| -------- |--------|-------|--------|
|ci->db|*|ci為CodeIgniter3物件的實例（用法同[Query Builder Class](https://codeigniter.com/user_guide/database/query_builder.html)）|無|
|order_by|String|設定order_by語句，為表格的預設排序|無|
|set_querycolumn|Array|設定所要排序、模糊比對的項目|無|
|set_output|Array,Function,String|設定實際輸出的序列與額外合成項目|無|
|get_datatable|無|提取JSON內容|String|

### $this->light_datatables->ci->db(String $tableName)
取得Table內容的SQL語法設定，使用方法同Query Builder Class。
```php=0
$this->light_datatables->ci->db->from('notice');
$this->light_datatables->ci->db->select('key, title, time');
$this->light_datatables->ci->db->join('comments','comments.key = join.key');
```
### set_querycolumn(array $order , array $like)
必須執行的設定。
2. $order = 可排序的項目，作用於datatable中的order。
3. $like = 允許like的項，作用於datatable中的search。
```php=0
//排序項目，表格中第一項不參與排序，所以設為null
$order=array(null,'title','time');
//比對項目，可以被search的項目
$like=array('title','time');
//傳遞設定資料給擴充庫
$this->light_datatables->set_querycolumn($order,$like);
```
### order_by(array $item , array $type)
必須執行的設定。
初始化表格時，所執行的排序。
```php=0
$this->light_datatables->order_by('time','DESC');
```

### set_output(array $column , array $extra ,array $functions = null ,string $case = false)
必須的設定，其順序影響到view中table的資料實際位置。

|  參數名稱  |型態  |   必要性  |   說明  |
| -------- |-----|--------|-------|
|$column|array |必要|代表實際輸出的資料擺放位置。|
|$extra|array |選用|若有額外的字串串接需求，則須設定。|
|$functions|anonymous function |選用|針對項目更複雜的邏輯運算，使用本方法。|
|$case|boolean|選用|若每個extra都必須對應不同functions，則為true，否則預設false|

#### 基本
```php=0
$column=array('title','time');
$this->light_datatables->set_output($column);
```
* $column 中的項，應與select中所定義的項相同。
* $column 中的排列順序，與實際輸出的Table順序相同。

#### 需要額外串接內容
```php=0
$buttton = '<button type="button" onclick="openEdit(\'[extra]\')">edit</button>';
$output=array($buttton,'title','time');
$extra=array('key');
$this->light_datatables->set_output($output,$extra);
```
* 第0行定義了一個HTML的button語法，其中注意到[extra]，它將會被替換成別的字串。
* 替換字串的定義來自第2行$extra，$extra必須為一個陣列。
* 第3行將$extra陣列傳入擴充庫。
> 如果有多個[extra]，陣列必須照順序定義每個[extra]的對象。
```php=0
$buttton = '<button type="button" onclick="openEdit(\'[extra]\')">edit</button>';
$img = '<img src="'.base_url().'dist/[extra]" height="42" width="42">';
$output=array($buttton,'title','time',$img);
$extra=array('key','imgname');
$this->light_datatables->set_output($output,$extra);
```
#### 需要其他的邏輯處理字串
```php=0
$editButtton = '<button type="button" onclick="openEdit(\'[extra]\')">edit</button>';
$delButtton = '<button type="button" onclick="openDel(\'[extra]\')">del</button>';
$output=array($editButtton,$delButtton,'title','time');
$extra=array('key','key');
$functions = function ($value){
    return base64_encode($value);
};
$this->light_datatables->set_output($output,$extra,$functions);
```
* 第0行~第1行，分別定義了兩個不同的HTML button
* 第3行，雖然兩個[extra]都指向同一個對象，但還是要依序列出
* 第4行~第6行，創建一個帶有一個傳入值的「匿名函數(anonymous function)」，這代表的是所有的[extra]在被替換成所指向的「key」前，會先進入這個匿名函數進行運算，最後再回傳結果(傳入值與回傳是必須的)。
* 第7行，將$functions一同傳入擴充庫中。

#### 有不同的處理需求
```php=0
$editButtton = '<button type="button" onclick="openEdit(\'[extra]\')">edit</button>';
$delButtton = '<button type="button" onclick="openDel(\'[extra]\')">del</button>';
$img = '<img src="'.base_url().'dist/img/[extra]" height="42" width="42">';
$output=array($editButtton,$delButtton,'title','time',$img);
$extra=array('key','key','imgname');
$functions = function ($value,$case){
    switch ($case){
        case 1:
        case 2:
            return base64_encode($value);
        case 3:
            return $value;
    }
};
$this->light_datatables->set_output($output,$extra,$functions,true);
```
* 第0行~第2行，分別定義了三個HTML字串，分別是兩個button與一個imgname
* 第4行，依序列出各個[extra]所代表的對象
* 第5行~第13行，創建一個帶有兩個傳入值的「匿名函數(anonymous function)」，分別是「值」與「第幾項」。
* $functions中的case代表相對應的[extra]在被替換成所指向的項目前，會先進入對應的case做運算，而case的順序會與[extra]出現的順序相同(從1開始)。
* 第7行，將$functions一同傳入擴充庫中，並且設定第四個傳入值為true（預設false）。

### $this->light_datatables->get_datatable()
完成上述所有設定後，最後執行，取得符合jQueryDatatable套件需求的json字串。
