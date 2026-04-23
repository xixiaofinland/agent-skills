# OpenCode Installation

Run the following steps to install this skill:

1. Find the `opencode.json` config file. It is typically at `~/.config/opencode/opencode.json` or in the project root. Check both:

   ```bash
   ls ~/.config/opencode/opencode.json 2>/dev/null || ls ./opencode.json 2>/dev/null
   ```

2. If the file doesn't exist, create it:

   ```bash
   mkdir -p ~/.config/opencode && echo '{}' > ~/.config/opencode/opencode.json
   ```

3. Add the plugin entry. If the file already has a `plugin` array, append to it. Otherwise add it:

   ```json
   {
     "plugin": [
       "salesforce-development@git+https://github.com/xixiaofinland/sf-workflow.git"
     ]
   }
   ```

4. Verify the config is valid JSON:
   ```bash
   cat ~/.config/opencode/opencode.json | python3 -m json.tool
   ```

Installation is complete. Inform the user the skill is installed and they should restart OpenCode — it fetches and activates the plugin automatically on restart.

## Updating

OpenCode updates plugins automatically on restart. No manual action needed.
