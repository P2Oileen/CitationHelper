# CitationHelper
Small Script displaying current Google Scholar citation count each time the shell is opened.

Google Scholar自搜小脚本，每次开启命令行即显示当前citation

## Result
For self-check 自搜的:
```bash
~ As opening 打开terminal时
✨Hi Ailin Huang, your Google Scholar citations✨: 110

~ check_me
✨Hi Ailin Huang, your Google Scholar citations✨: 110
```

For multi-check 多人查询:
```bash
~ As opening 打开terminal时
✨Hi Ailin Huang, your Google Scholar citations✨: 110
Zhewei Huang's citations: 902
Xiaotao Hu's citations: 253

~ check_us
✨Hi Ailin Huang, your Google Scholar citations✨: 110
Zhewei Huang's citations: 902
Xiaotao Hu's citations: 253

~ check_me
✨Hi Ailin Huang, your Google Scholar citations✨: 110
```

## Usage

### Self check
First, obtain your GOOGLE_SCHOLAR_USER_ID, which is the string found on your profile page URL.

首先需要获取一下自己的GOOGLE_SCHOLAR_USER_ID，即你的scholar主页的这一串
![image](https://github.com/user-attachments/assets/c1cbcff7-ea75-4e65-9afc-8e3e4ab2447f)



Open `~/.bashrc` or `~/.zshrc` file and paste the following code at the end of the file:

打开`~/.bashrc` 或者 `~/.zshrc`，把下面这段粘贴到文件末尾。

Replace GOOGLE_SCHOLAR_USER_ID with your actual Google Scholar user ID.

把GOOGLE_SCHOLAR_USER_ID换成自己的

```bash
# Change this user id 把这里的id换成你自己的
GOOGLE_SCHOLAR_USER_ID="LM7RNL4AAAAJ"

check_me() {
  response=$(curl -s "https://scholar.google.com/citations?user=${GOOGLE_SCHOLAR_USER_ID}&hl=en")

  # Get Citation
  citations=$(echo $response | grep -o '<td class="gsc_rsb_std">[0-9]*' | grep -o '[0-9]*')
  citations=$(echo $citations | head -n 1)
  if [ -z "$citations" ]; then
    citations="0"
  fi

  # Get Name
  name=$(echo $response | grep -o '<div id="gsc_prf_in">[^<]*</div>' | grep -o '>[^<]*<')
  name=${name//[><]/}
  if [ -z "$name" ]; then
    name="unknown"
  fi

  echo "✨Hi $name, your Google Scholar citations✨: $citations"
}

check_me
```

Finally, apply the changes by sourcing the file. Run `source ~/.bashrc` or `source ~/.zshrc`, depending on which shell you are using.

最后应用修改，`source ~/.bashrc`或者`source ~/.zshrc`。

### Multi check
If you want to display the citation counts for other researchers alongside your own, use scripts below.

如果你要查其他圈内好友就用下面这个脚本

```
MY_USER_ID="LM7RNL4AAAAJ" # Replace with your ID 这里是你的ID
FRIEND_USER_IDS=("zJEkaG8AAAAJ" "PRcnqk4AAAAJ")  # Replace with other researchers' ID 添加其他用户ID

check_me() {
  local user_id=${1:-$MY_USER_ID}
  response=$(curl -s "https://scholar.google.com/citations?user=${user_id}&hl=en")

  # Get citation
  citations=$(echo $response | grep -o '<td class="gsc_rsb_std">[0-9]*' | grep -o '[0-9]*')
  citations=$(echo $citations | head -n 1)
  if [ -z "$citations" ]; then
    citations="0"
  fi

  # Get name
  name=$(echo $response | grep -o '<div id="gsc_prf_in">[^<]*</div>' | grep -o '>[^<]*<')
  name=${name//[><]/}
  if [ -z "$name" ]; then
    name="unknown"
  fi

  # For self check, return "✨Hi $name, your Google Scholar citations✨: $citations"
  # others return "$name's citations: $citations"
  if [[ "$user_id" == "$MY_USER_ID" ]]; then
    echo "✨Hi $name, your Google Scholar citations✨: $citations"
  else
    echo "$name's citations: $citations"
  fi
}

check_us() {
  check_me
  for user_id in "${FRIEND_USER_IDS[@]}"; do
    check_me "$user_id"
  done
}

check_us
```

### Too Large html file
If you meet an error that the curl html file is too large and your name could not be grep from it, we provide additional code to save your html file to a local temp and pcregrep it.

如果你的Google scholar html文件太大了导致无法匹配到你的姓名，我们提供了以下代码来保存该文件到一个temp区然后用pcregrep来处理。

```
# The following code is helped with windsurf and Claude 3.5 sonnet.
# Change this user id 把这里的id换成你自己的
GOOGLE_SCHOLAR_USER_ID="LM7RNL4AAAAJ"

check_me() {
  # Save response to a temporary file instead of variable
  temp_file=$(mktemp)
  curl -s "https://scholar.google.com/citations?user=${GOOGLE_SCHOLAR_USER_ID}&hl=en" > "$temp_file"

  # Get Citation using the file with increased buffer size
  citations=$(pcregrep --buffer-size=1M -o '<td class="gsc_rsb_std">[0-9]*' "$temp_file" | pcregrep -o '[0-9]*' | head -n 1)
  if [ -z "$citations" ]; then
    citations="0"
  fi

  # Get Name using the file with increased buffer size
  name=$(pcregrep --buffer-size=1M -o '<div id="gsc_prf_in">[^<]*</div>' "$temp_file" | sed 's/<[^>]*>//g')
  if [ -z "$name" ]; then
    name="unknown"
  fi

  # Clean up
  rm "$temp_file"

  echo "✨Hi $name, your Google Scholar citations✨: $citations."
}

check_me
```

Have Fun!

P.S. This project was developed with the assistance of <a href="https://yuewen.cn/">跃问/Yuewen</a>, an AI assistant created by our group <a href="https://github.com/stepfun-ai">StepFun-ai</a>.
