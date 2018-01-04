---
title: 当nz-checkbox-group多选框组遇上必选校验
tags: Angular2,ng-zorro-antd,checkbox
grammar_cjkRuby: true
---
今天表单中用到ng-zorro-antd组件的多选框nz-checkbox-group，最开始用的是响应式表单的验证+响应式表单的验证，结果总是无法达到预期效果。
## 初始代码及问题现象：
问题.Html
```
<div nz-form-item nz-row>
        <div nz-form-label nz-col [nzSpan]="4">
          <label for="one" nz-form-item-required>复选测试</label>
        </div>
         <div nz-form-control nz-col [nzSpan]="10" [nzValidateStatus]="getFormControl('one')">
          <nz-checkbox-group formControlName="one"  [(ngModel)]="oneOption" ></nz-checkbox-group>
          <div nz-form-explain *ngIf="getFormControl('one').dirty&&getFormControl('one').hasError('required')">通知范围必选</div>
        </div>
      </div>
```
问题.ts
这里仅列出关键代码部分
```
validateForm: FormGroup;
oneOption: any;
constructor(
    private fb: FormBuilder,
 ){}
ngOnInit() {
    this.oneOption = [
    { label: 'Apple', value: 'Apple', checked: true },
    { label: 'Pear', value: 'Pear', checked: false },
    { label: 'Orange', value: 'Orange', checked: false },
    ]
    this.validateForm = this.fb.group({
         one: [null, [Validators.required]],
    })
}
  getFormControl(name) {
    return this.validateForm.controls[name];
  }
```
为了找问题，在提交方法_submitForm（）中添加了几个打印
```
console.log(this.getFormControl('one').dirty); // 1
console.log(this.getFormControl('one').hasError('required')); // 2
console.log(this.validateForm.value.one); // 3
console.long(this.validateForm.invalid);// 4
```
结果发现
- 初始时：1、true，2、false，3、oneOption中的值，4、false
- 选择一个选项后：1、false，2、false，3、oneOption中的值+选中的value，4、false

从而始终无法触发显示 “通知范围必选”
## 第一次尝试
最开始尝试是将```this.validateForm.value.scopes```在提交时先赋值为[]，再检测checked状态，赋值。
```
this.validateForm.value.scopes = [];
for(const i in this.oneOption){
	if(this.oneOption[i].checked){
		this.validateForm.value.scopes.push(this.oneOption[i].value);
	}
}
```
这样写 ，```this.validateForm.value.one```是达到预期了，但提示效果依旧没出来。
继续探索，看到```getFormControl('one').hasError('required')```，既然有has，有没有set一类的？打了一下发现还真有一个this.getFormControl('one').setErrors()的方法，于是在上面的基础上想到这样一个方式：
```
if(this.validateForm.value.scopes.length == 0){
	this.getFormControl('one').setErrors(Validators.required);
}
```
发现一点用没有，搜索后找到另一种变形：
```
if(this.validateForm.value.scopes.length == 0){
	this.getFormControl('one').setErrors({'required':true});
}
```
发现上面的2可以变成true了，但1始终是false，导致无效。
## 再次尝试
到这曾一度想过放弃然后自己用原始方式写，再一想到原始方式还要自己考虑样式什么的，作为一个有着css恐惧症的Java程序猿我决然地选择了硬着头皮在啃会儿。

在刷了n+1遍ng-zorro-antd的官方文档的表单部分后，在“自定义异步校验”中看到这样一句话 :
>当使用 响应式表单(Reactive Form) 时，<nz-form-control> 的 nzValidateStatus 会自动从 NgControl 中获取数据，也**可以手动指定特定**的 NgControl
组件将表单校验函数的校验过程和异步返回的结果显示对应的error | validating(pending) | warning | success状态，具体使用方式建议参照本demo

本着死马当活马医的心点开里面的dome，仔细看了下，同时在实例上试了一下，发现这不正是梦寐以求的咩。于是有了如下的终极解决方案：

问题.html中不用做修改。

问题.ts修改如下：

```
//因为不想在提交方法_submitForm（）再循环一遍获取多选结果，就只好在这先定义一个临时的用于存储选择结果。
 selectedOne: any = [];
 
 validateForm: FormGroup;
oneOption: any;
constructor(
    private fb: FormBuilder,
 ){}
ngOnInit() {
    this.oneOption = [
    { label: 'Apple', value: 'Apple', checked: true },
    { label: 'Pear', value: 'Pear', checked: false },
    { label: 'Orange', value: 'Orange', checked: false },
    ]
    this.validateForm = this.fb.group({
         one: [null, [this.onesValidator]],//修改校验规则为下面自定义的onesValidator
    })
}
  getFormControl(name) {
    return this.validateForm.controls[name];
  }
   _submitForm() {
   	this.validateForm.value.one = this.selectedOne;
   }
 //创建自定义校验规则onesValidator，用于复选框组校验时调用。
 onesValidator= (control: FormControl): { [s: string]: boolean } => {
    this.selectedOne = [];

    for (const i in control.value) {
      if (control.value[i].checked) {
        this.selectedOne.push(control.value[i]);
      }
    }
    if (this.selectedOne.length == 0) {
      return { required: true};
    }
  };
```
改完后测试，nice~一切走上正轨。至此圆满结束，完结撒花。