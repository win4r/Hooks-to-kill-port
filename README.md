# Hooks-to-kill-port

步骤 1：创建智能脚本
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
chmod +x ~/.claude/kill-port.sh

步骤 2：修改 settings.json
{
  "model": "claude-opus-4-5-20251101",
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
  },
  "enabledPlugins": {
    "pyright@claude-code-lsps": true,
    "pyright-lsp@claude-plugins-official": true
  }
}

步骤 3：重启 Claude Code
/exit
claude

