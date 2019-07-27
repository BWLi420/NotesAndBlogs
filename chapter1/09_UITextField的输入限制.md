# 输入金额的限制

1. 金额只能包含数字 0 ~ 9 和 小数点；
2. 首位不能是小数点；
3. 小数点只能存在一个；
4. 首位为 0 时，第二位必须是小数点；
5. 小数点后面最多两位。

- 这里主要使用了 textField 的以下代理方法

```objective-c
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string
```

- 下面是具体代码，内含详细注释

```objective-c
// textField输入金额的限制
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    
    //金额的最大长度，自行调节
    if (textField.text.length > 10) {
        return range.location < 11;
    }else{
        
        //记录是否有小数点
        BOOL isHaveDian = YES;
        if ([textField.text rangeOfString:@"."].location == NSNotFound) {
            isHaveDian=NO;
        }
        
        if ([string length] > 0) {
            unichar single = [string characterAtIndex:0];//当前输入的字符
            
            if ((single >= '0' && single <= '9') || single == '.') {//数据格式正确
                
                //首字母不能为小数点
                if([textField.text length] == 0){
                    if(single == '.'){
                        [textField.text stringByReplacingCharactersInRange:range withString:@""];
                        return NO;
                    }
                }
                
                //首位为0时，只能输入小数点
                if([textField.text length] == 1 && [textField.text isEqualToString:@"0"]){
                    if(single != '.'){
                        [textField.text stringByReplacingCharactersInRange:range withString:@""];
                        [RKDropdownAlert title:@"提示" message:@"首位为0时，只能输入小数点"];
                        return NO;
                    }
                }
                
                if (single == '.') {
                    
                    //text中还没有小数点
                    if(!isHaveDian) {
                        
                        isHaveDian = YES;
                        return YES;
                    }else {
                        
                        //只能有一个小数点
                        [textField.text stringByReplacingCharactersInRange:range withString:@""];
                        return NO;
                    }
                    
                }else {
                    
                    //存在小数点
                    if (isHaveDian) {
                        
                        //判断小数点的位数
                        NSRange ran = [textField.text rangeOfString:@"."];
                        NSInteger tt = range.location - ran.location;
                        if (tt <= 2){
                            return YES;
                        }else{
                            
                            //小数点后最多两位
                            return NO;
                        }
                        
                    }else {
                        
                        return YES;
                    }
                }
                
            }else{
                
                //输入的数据格式不正确 -> 金额只能输入数字和小数点
                [textField.text stringByReplacingCharactersInRange:range withString:@""];
                return NO;
            }
            
        }else {
            return YES;
        }
    }
}
```

# 输入数字的限制

```objective-c
#define NUMBERS @"0123456789"

- (BOOL)textField:(UITextField*)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString*)string
{
    NSCharacterSet *cs;
    cs = [[NSCharacterSet characterSetWithCharactersInString:NUMBERS] invertedSet];
    NSString *filtered = [[string componentsSeparatedByCharactersInSet:cs] componentsJoinedByString:@""];
    BOOL basicTest = [string isEqualToString:filtered];
    if(!basicTest) {
        
        NSLog(@"提示：只能输入数字！");
        return NO;
    }
    
    return YES;
}
```

