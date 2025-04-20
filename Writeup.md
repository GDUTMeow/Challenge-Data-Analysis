# Data-Analysis

**所有数据均为随机生成**

## 校验规范

> - 编号：纯数字，最后应该从小到大排列
> - 用户名：由数字、字母组成
> - 密码：密码 hash 值为 32 位小写 md5 值
> - 姓名：由全中文组成
> - 性别：只能为 `男` 或者 `女`，且身份证号中代表性别的那一位要对应上
> - 出生日期：由 8 位数字组成，和身份证号中的出生日期码保持一致
> - 身份证号：应该符合国家对于身份证号的校验规则，本题中提供的身份证号均符合规则
> - 手机号码为 11 位 10 进制数字字符串，前三位的号段限定在以下的集合中
>   - `734, 735, 736, 737, 738, 739, 747, 748, 750, 751, 752, 757, 758, 759, 772, 778, 782, 783, 784, 787, 788, 795, 798, 730, 731, 732, 740, 745, 746, 755, 756, 766, 767, 771, 775, 776, 785, 786, 796, 733, 749, 753, 773, 774, 777, 780, 781, 789, 790, 791, 793, 799`

## 题目要求

你应该将每一行都变成与表头相同的顺序排列，表头顺序为 `编号, 用户名, 密码, 姓名, 性别, 出生日期, 身份证号, 手机号码`，对应 `number` `username` `password` `name` `gender` `birth` `id` `phone`

例如，假设你拿到的数据为（下面数据为瞎编的）

```
编号,用户名,密码,姓名,性别,出生日期,身份证号,手机号码
Luminoria,79811451419,114514200002291919,20000229,小猪佩奇,1,男,e10adc3949ba59abbe56e057f20f883e
20050330,2,c4d038b4bed09fdb1471ef51ec3a32cd,7529876543,男,191981200503301155,刻晴,KeqingMoe
```

则最后你应该把数据变成这样

```
number,username,password,name,gender,birth,id,phone
1,Luminoria,e10adc3949ba59abbe56e057f20f883e,小猪佩奇,男,20000229,114514200002291919,79811451419
2,KeqingMoe,c4d038b4bed09fdb1471ef51ec3a32cd,刻晴,男,20050330,191981200503301155,7529876543

```

将数据**以 UTF-8 编码方式**保存后，确保**最后含有一行空行**，将文件内容进行 md5 运算，最后得到的值加上 `flag{}` 包裹即为最后的答案

## 注意

- 你的换行应该使用 `\n`，你可以在调试的时候使用 `print(repr(data))` 来确认这一点
- 在你进行 md5 运算的文件内容中，最后有一个空行，就例如上面给你处理后的例子中，最后一行有一个空行
- 你进行 md5 运算的文件内容中，你需要注意不要忘掉首行的表头

## 题解

本题数据集来自 2024 年羊城杯初赛

```python
import re
from tqdm import tqdm
import csv

with open("person_data.csv", encoding="utf8") as f:
    data = f.readlines()


def getIDNumber(line):	# 限定身份证号的格式为17位数字+X或者18位纯数字
    match = re.search(r"\b(?:\d{17}[0-9X]|\d{15})\b", line)
    return match[0]


def getPassword(line):	# 限制密码为连续的32位字母数字组合
    match = re.search(r"[a-f0-9]{32}", line)
    return match[0]


def getName(line):	# 限定名字为两个中文字符以上的连续中文
    match = re.search(r"[\u4e00-\u9fa5]{2,}", line)
    return match[0]


def getGender(line):	# 限定性别为男或者女，且中文字符的前面和后面至少有一个逗号（避免名字中含有男、女二字造成匹配出错）
    match = re.search(r"(?<!\w)(男|女)(?!\w)(?=(?:[^,]*,){0,1}(?=,|$))", line)
    return match[0]


def getBirth(idNumber):	# 从身份证中获取生日
    birth = idNumber[6:14]
    return birth


def getPhone(line, idNumber, passwd):	# 除去身份证号、密码（32位hash值存在连续10或11位数字，所以要去掉）后，剩下还能够匹配上的则为手机号
    data = line.replace("\n", "").replace(idNumber, "").replace(passwd, "")
    pattern = r"(734|735|736|737|738|739|747|748|750|751|752|757|758|759|772|778|782|783|784|787|788|795|798|730|731|732|740|745|746|755|756|766|767|771|775|776|785|786|796|733|749|753|773|774|777|780|781|789|790|791|793|799)\d{7,8}"
    match = re.search(pattern, data)
    return match[0]


def getUsername(line, number, passwd, name, gender, birth, idNumber, phone):	# 去掉上面已经匹配掉的任何内容，剩下的为用户名
    data: list = line.replace("\n", "").split(",")
    toRemove = [number, passwd, name, gender, birth, idNumber, phone]
    for item in toRemove:
        data.remove(str(item))
    return data


def process_data(data):
    # 打开 CSV 文件以写入数据
    with open("output.csv", "w", newline="", encoding="utf-8") as csvfile:
        fieldnames = [
            "number",
            "username",
            "password",
            "name",
            "gender",
            "birth",
            "id",
            "phone",
        ]
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

        # 写入表头
        writer.writeheader()

        # 遍历数据
        for i in tqdm(range(len(data))):
            if i == 0:
                continue  # 第一行为表头，执行跳过

            number = i
            idNumber = getIDNumber(data[i])
            passwd = getPassword(data[i])
            name = getName(data[i])
            gender = getGender(data[i])
            birth = str(getBirth(idNumber))
            phone = getPhone(data[i], idNumber, passwd)
            username = getUsername(
                data[i], number, passwd, name, gender, birth, idNumber, phone
            )

            # 写入数据行
            writer.writerow(
                {
                    "number": number,
                    "username": username[0],
                    "password": passwd,
                    "name": name,
                    "gender": gender,
                    "birth": birth,
                    "id": idNumber,
                    "phone": phone,
                }
            )


if __name__ == "__main__":
    process_data(data)
```

`flag{518a8b1f87c23d94c1be372efd0f0f30}`