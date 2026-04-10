# format()
num = 1234.56789
print(format(num, 'f'))      # 默认：1234.567890
print(format(num, '.2f'))    # 两位小数：1234.57
print(format(num, '.0f'))    # 无小数：1235
print(format(num, 'e'))      # 科学计数法：1.234568e+03
print(format(num, '.3e'))    # 科学计数法3位小数：1.235e+03
print(format(num, 'g'))      # 一般格式：1234.57
print(format(num, '.6g'))    # 6位有效数字：1234.57
print(format(num, '.10g'))   # 10位有效数字：1234.56789

# rstrip()
去除后缀
### 单独使用rstrip('0')
s = "123.45600"
print(s.rstrip('0'))  # 输出: "123.456"

s2 = "100.00"
print(s2.rstrip('0'))  # 输出: "100."

s3 = "123000"
print(s3.rstrip('0'))  # 输出: "123"

### rstrip('0').rstrip('.')联用
去除末尾的0，然后去除可能留下的小数点
s = "123.4500"
print(s.rstrip('0').rstrip('.'))  # 输出: "123.45"

