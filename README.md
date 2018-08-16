# labelGroupAndStreamSwift
使用按钮实现单选，多选，分组，标签流功能
![图片](https://upload-images.jianshu.io/upload_images/3601465-dbb06fcc73fcb25c.gif?imageMogr2/auto-orient/strip)
###### 基本属性
```
//MARK:----publish
/// 组高度
public var titleLabHeight = 30
/// 组标题字体
public var titleTextFont : UIFont = .boldSystemFont(ofSize: 14)
/// 组标题 字体颜色
public var titleTextColor : UIColor = .black
/// 显示按钮的高度
public var content_height = 30
/// 上下按钮之间的间距
public var content_y = 10
/// 左右按钮之间的间距
public var content_x = 10
/// title默认颜色
public var content_norTitleColor : UIColor = .white
/// title选中颜色
public var content_selTitleColor : UIColor = .white
/// 背景默认原色
public var content_backNorColor : UIColor = .gray
/// 背景选中颜色
public var content_backSelColor : UIColor = .orange
/// 字体大小
public var content_titleFont : UIFont = .systemFont(ofSize: 12)
/// 圆角
public var content_radius : Int = 8
/// 是否单选，默认 true 是单选
public var isSingle : Bool = true
/// 是否设置默认选中 默认 true 默认选中
public var isDefaultChoice : Bool = true
/// isDefaultChoice 为true时 改属性有效，默认为 0
public var defaultSelIndex : Int = 0
/// isDefaultChoice 为true时 改属性有效 defaultSelIndex 属性无效，为每各组设置单选选项
public var defaultSelSingleIndeArr : Array = Array<Any>()
/// 为每个组设置单选或多选，设置该属性时 isSingle 参数无效, 0 = 多选， 1 = 单选
public var defaultGroupSingleArr = Array<Int>(){
didSet{
for value in defaultGroupSingleArr {
if !(value == 0 || value == 1){
assert((value == 0 || value == 1), "defaultGroupSingleArr的值只能是 0 和 1")
}
}
}
}
/// isDefaultChoice 为true时 该属性有效，设置每组默认选择项，可传数组
public var defaultSelIndexArr = Array<Any>() {
didSet{
for (index,value) in defaultSelIndexArr.enumerated() {
//当Value的类型是数组时则 isSingle 为false 多选
if value is Array<Any>{
if !defaultGroupSingleArr.isEmpty{
//Value 是数组，并 defaultGroupSingleArr 不为空
defaultGroupSingleArr[index] = 0
}
isSingle = false
}
}
}
}
/// 闭包传值，传出所有选择的值和groupid
public var confirmReturnValueClosure : ((Array<Any>,Array<Any>) -> Void)?
/// 闭包传值，传出当前选中的值
public var currentSelValueClosure : ((String,Int,Int) -> Void)?
/// 代理
public weak var delegate : CBGroupAndStreamViewDelegate?
```
###### 文字自适应
```
let but_width = calcuateLabSizeWidth(str: value as! String, font:content_titleFont, maxHeight: CGFloat(content_height)) + 20
//计算每个button的 X
margin_x = CGFloat(alineButWidth) + CGFloat(content_x)
//计算一行的宽度
alineButWidth = CGFloat(content_x) + but_width + CGFloat(alineButWidth)
//判断是否需要换行
if alineButWidth >= self.frame.size.width{
margin_x = CGFloat(content_x)
alineButWidth = margin_x + but_width
content_totalHeight = current_rect.size.height + current_rect.origin.y + CGFloat(content_x)
}
//            print("margin_x = \(margin_x)")
sender.frame = CGRect(x: margin_x, y: content_totalHeight, width: but_width, height: CGFloat(content_height))
//临时保存frame，以进行下一次坐标计算
current_rect = sender.frame
```

###### 设置默认选中
```
//MARK:---设置默认---单选
private func setDefaultSingleSelect(index : Int , groupId : Int ,value : String, sender : UIButton, content : Array<Any>){
//单选
let valueStr = "\(index)/\(value)"
if defaultSelSingleIndeArr.isEmpty{
assert( !(defaultSelIndex  > content.count - 1), "在groupId = \(groupId) 设置默认选中项不能超过\(content.count - 1)")
if index == defaultSelIndex{
sender.isSelected = true
sender.backgroundColor = content_backSelColor
saveSelButValueArr[groupId] = valueStr
}
}else{
assert(!((defaultSelSingleIndeArr[groupId] as? Int)! > content.count - 1), "在groupId = \(groupId) 设置默认选中项不能超过\(content.count - 1)")
if index == defaultSelSingleIndeArr[groupId] as? Int{
sender.isSelected = true
sender.backgroundColor = content_backSelColor
saveSelButValueArr[groupId] = valueStr
}
}
saveSelGroupIndexeArr[groupId] = String(groupId)
}
//MARK:---设置默认---多选
private func setDefaultMultipleSelect(index : Int , groupId : Int ,value : String, sender : UIButton, content : Array<Any>) -> Array<Any>{
let content = defaultSelIndexArr[groupId]
var tempSaveSelIndexArr = Array<Any>()
if content is Int{
if index == content as! Int{
sender.isSelected = true
sender.backgroundColor = content_backSelColor
tempSaveSelIndexArr.append("\(index)/\(value)")
}
}
if content is Array<Any>{
for contenIndex in content as! Array<Any>{
if index == contenIndex as! Int{
sender.isSelected = true
sender.backgroundColor = content_backSelColor
tempSaveSelIndexArr.append("\(index)/\(value)")
continue
}
}
}
saveSelGroupIndexeArr[groupId] = String(groupId)
return tempSaveSelIndexArr
}
```
###### 单选，多选
```
//MARK:---单选
private func singalSelectEvent(sender : UIButton){
var valueStr : String = ""
let tempDetailArr = dataSourceArr[sender.tag / 100] as! Array<Any>
if sender.isSelected {
for (index, _) in tempDetailArr.enumerated(){
if index + 1 == sender.tag % 100{
sender.isSelected = true
sender.backgroundColor = content_backSelColor
continue
}
let norSender = scrollView.viewWithTag((sender.tag / 100) * 100 + index + 1) as! UIButton
norSender.isSelected = false
norSender.backgroundColor = content_backNorColor
}
valueStr = "\(sender.tag % 100 - 1)/\(tempDetailArr[sender.tag % 100 - 1])"
//闭包传值
if currentSelValueClosure != nil {
currentSelValueClosure!(valueStr,sender.tag % 100 - 1,sender.tag / 100)
}
//代理传值
delegate?.currentSelValueWithDelegate?(valueStr: valueStr, index: sender.tag % 100 - 1, groupId: sender.tag / 100)
}else{
sender.backgroundColor = content_backNorColor
}
//保存选中的值
saveSelButValueArr[sender.tag / 100] = valueStr
//保存groupId
saveSelButValueArr[sender.tag / 100] as! String == "" ? (saveSelGroupIndexeArr[sender.tag / 100] = "") : (saveSelGroupIndexeArr[sender.tag / 100] = String(sender.tag / 100))

}

//MARK:---多选
private func multipleSelectEvent(sender : UIButton){
var valueStr = ""
var tempSaveArr = Array<Any>()
if ((saveSelButValueArr[sender.tag / 100]) is Array<Any>){
tempSaveArr = saveSelButValueArr[sender.tag / 100] as! Array<Any>
}else{
tempSaveArr.append(saveSelButValueArr[sender.tag / 100])
}

let tempDetailArr = dataSourceArr[sender.tag / 100] as! Array<Any>
valueStr = "\(sender.tag % 100 - 1)/\(tempDetailArr[sender.tag % 100 - 1])"
if sender.isSelected {
sender.backgroundColor = content_backSelColor
//不存在相同的元素
tempSaveArr.append(valueStr)
//闭包传值
if currentSelValueClosure != nil {
currentSelValueClosure!(valueStr,sender.tag % 100 - 1,sender.tag / 100)
}
//代理传值
delegate?.currentSelValueWithDelegate?(valueStr: valueStr, index: sender.tag % 100 - 1, groupId: sender.tag / 100)
}else{
sender.backgroundColor = content_backNorColor
//获取元素的下标
let index : Int = tempSaveArr.index(where: {$0 as! String == valueStr})!
tempSaveArr.remove(at: index)
}

saveSelButValueArr[sender.tag / 100] = tempSaveArr
tempSaveArr.isEmpty ? (saveSelGroupIndexeArr[sender.tag / 100] = "") : (saveSelGroupIndexeArr[sender.tag / 100] = String(sender.tag / 100))

}
```
###### 计算文字宽度
```
//MARK:---计算文字宽度
private func calcuateLabSizeWidth(str : String, font : UIFont, maxHeight : CGFloat) -> CGFloat{
let attributes = [kCTFontAttributeName: font]
let norStr = NSString(string: str)
let size = norStr.boundingRect(with: CGSize(width: CGFloat(MAXFLOAT), height: maxHeight), options: .usesLineFragmentOrigin, attributes: attributes as [NSAttributedStringKey : Any], context: nil)
return size.width
}
```

###### 使用方法
```
let titleArr = ["关系","花","节日","枝数"]
let contentArr = [["恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他","恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他"],["玫","百合","康乃馨","郁金香","扶郎","马蹄莲"],["情人节","母亲节","圣诞节","元旦节","春节","恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他"],["9枝","100000000枝","11枝","21枝","33枝","99枝","99999999枝以上","恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他","恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他","恋人","朋友朋友朋友朋友朋友朋友","亲人恩师恩师","恩师恩师","病人","其他"]]

labGroup = CBGroupAndStreamView.init(frame: CGRect(x: 0, y: 0, width: UIScreen.main.bounds.size.width, height: UIScreen.main.bounds.size.height))
labGroup.titleTextFont = .systemFont(ofSize: 14)
labGroup.titleLabHeight = 30;
labGroup.titleTextColor = .red
labGroup.isSingle = true
//        labGroup.defaultSelIndex = 1
//        labGroup.defaultSelSingleIndeArr = [1,1,0,0]
//使用该参数则默认为多选 isSingle 无效 defaultSelSingleIndeArr 设置无效
labGroup.defaultSelIndexArr = [[0,5,8,3,2],1,0,3]
//分别设置每个组的单选与多选
labGroup.defaultGroupSingleArr = [0,1,1,0]
labGroup.setDataSource(contetnArr: contentArr, titleArr: titleArr)
labGroup.delegate = self
self.view.addSubview(labGroup)

//闭包接收值
labGroup.confirmReturnValueClosure = {
(selArr,groupIdArr) in
//            print(selArr)
}
labGroup.currentSelValueClosure = {
(valueStr,index,groupId) in
//            print("\(valueStr) index = \(index), groupid = \(groupId)")
}

//代理
extension ViewController : CBGroupAndStreamViewDelegate{

func currentSelValueWithDelegate(valueStr: String, index: Int, groupId: Int) {
print("\(valueStr) index = \(index), groupid = \(groupId)")
}

func confimrReturnAllSelValueWithDelegate(selArr: Array<Any>, groupArr: Array<Any>) {
print(selArr)
}
}

```

###### [swift版简书](https://www.jianshu.com/p/c7de7f8de3ba)
###### [OC版简书](https://www.jianshu.com/p/05813eea7995)







