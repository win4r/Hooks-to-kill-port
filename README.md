# Claude Code 智能端口管理 Hook

### 我的频道：https://www.youtube.com/@AIsuperdomain

在使用 Claude Code 启动 Node.js 项目时，自动关闭占用端口的旧进程，避免 "port already in use" 错误。

## 功能特点

- **智能检测**：只在执行 npm/yarn/pnpm/node 启动命令时触发
- **多端口支持**：同时监控多个常用端口（3000, 3001, 5173, 8080 等）
- **无感知运行**：作为 PreToolUse Hook 在命令执行前自动运行
- **全局配置**：一次配置，所有项目生效


### 步骤 1：创建智能脚本
```
cat > ~/.claude/kill-port.sh << 'EOF'
#!/bin/bash
# 检测是否是启动服务命令
INPUT="$1"
if echo "$INPUT" | grep -qE "npm|node|yarn|pnpm"; then
    for port in 3000 3001 5173 8080; do
        pid=$(lsof -t -i:$port 2>/dev/null)
        [ -n "$pid" ] && kill -9 $pid && echo "已关闭端口 $port (PID: $pid)"
    done
fi
exit 0
EOF
```

```
chmod +x ~/.claude/kill-port.sh
```


### 步骤 2：修改 settings.json
```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/kill-port.sh \"$CLAUDE_TOOL_INPUT\"",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### 步骤 3：重启 Claude Code
```
/exit
claude
```

