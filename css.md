# css

/**
 * 书籍文章解读 - 古风文墨主题
 * 特征：宣纸底色、书法标题、朱砂点缀、优雅排版  
 */  
container {  
    background-color: #f8f4e9; /* 宣纸底色 */  
    padding: 2em;  
    font-family: "Noto Serif SC", "SimSun", serif; /* 优先使用思源宋体 */  
    line-height: 1.8;  
    color: #333;  
}

/* 标题样式 - 书法风格 */  
h1 {  
    font-family: "STXingkai", "KaiTi", cursive; /* 优先使用华文行楷 */  
    color: #5c3317; /* 深褐色 */  
    font-size: 2.2em;  
    border-bottom: 1px solid #d4a76a; /* 金色分隔线 */  
    padding-bottom: 0.3em;  
    text-align: center;  
    letter-spacing: 0.1em;  
}

h2 {  
    font-family: "STKaiti", "KaiTi", cursive;  
    color: #7a4a3a;  
    font-size: 1.8em;  
    border-left: 4px solid #c12c1f; /* 朱砂红竖线 */  
    padding-left: 0.8em;  
    margin-top: 1.5em;  
}

h3 {  
    color: #8c5e3e;  
    font-size: 1.4em;  
    font-weight: 600;  
    text-decoration: underline;  
    text-decoration-color: #d4a76a;  
    text-underline-offset: 0.3em;  
}

h4, h5, h6 {  
    color: #5c3317;  
    font-size: 1.1em;  
    font-style: italic;  
}

/* 图片样式 - 古籍插图效果 */  
image {  
    border: 1px solid #d4a76a;  
    box-shadow: 0 4px 8px rgba(92, 51, 23, 0.2);  
    margin: 1em auto;  
    display: block;  
    max-width: 90%;  
}

/* 引用样式 - 仿古籍批注 */  
blockquote {  
    background-color: #f1e8d6;  
    border-left: 4px solid #c12c1f;  
    padding: 1em 1.5em;  
    margin: 1em 0;  
    font-style: italic;  
    color: #5c3317;  
}

blockquote_p {  
    margin: 0.5em 0;  
    line-height: 1.6;  
}

/* GFM 笔记样式 */  
blockquote_note {  
    background-color: #f0f7ff;  
    border-left: 4px solid #4a6ea9;  
}

blockquote_tip {  
    background-color: #f0fff4;  
    border-left: 4px solid #2e7d32;  
}

blockquote_warning {  
    background-color: #fff8e1;  
    border-left: 4px solid #ff8f00;  
}

blockquote_caution {  
    background-color: #ffebee;  
    border-left: 4px solid #c62828;  
}

/* GFM 标题样式 */  
blockquote_title {  
    font-weight: bold;  
    margin-bottom: 0.5em;  
}

blockquote_title_note { color: #4a6ea9; }  
blockquote_title_tip { color: #2e7d32; }  
blockquote_title_warning { color: #ff8f00; }  
blockquote_title_caution { color: #c62828; }

/* 段落样式 */  
p {  
    text-align: justify;  
    text-justify: inter-character;  
    margin: 1em 0;  
    hyphens: auto;  
}

/* 分割线 - 仿古籍装订线 */  
hr {  
    border: none;  
    height: 1px;  
    background-image: linear-gradient(to right, transparent, #d4a76a, transparent);  
    margin: 2em 0;  
}

/* 行内代码 */  
codespan {  
    background-color: #f1e8d6;  
    color: #c12c1f;  
    padding: 0.2em 0.4em;  
    border-radius: 3px;  
    font-family: "Courier New", monospace;  
}

/* 链接样式 - 朱砂红 */  
link {  
    color: #c12c1f;  
    text-decoration: none;  
    border-bottom: 1px dotted #c12c1f;  
}

wx_link {  
    color: #c12c1f;  
    font-weight: bold;  
}

/* 列表样式 */  
ol, ul {  
    padding-left: 2em;  
}

li {  
    margin-bottom: 0.5em;  
    position: relative;  
}

/* 代码块 - 仿雕版印刷 */  
code_pre {  
    background-color: #2e2e2e;  
    color: #f8f8f2;  
    padding: 1em;  
    border-radius: 4px;  
    overflow-x: auto;  
    font-family: "Courier New", monospace;  
    border-left: 4px solid #c12c1f;  
}

code {  
    font-family: "Courier New", monospace;  
}

/* 强调文本 */  
strong {  
    color: #8c2e0b;  
    font-weight: bold;  
}
