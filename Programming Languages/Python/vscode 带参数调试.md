# vscode 带参数调试 Python

有时候需要调试 Python, 但是原始命令带了参数, 无法通过直接点击 VSCode 右上角的 `Debug Python File` 来实现.

## 简单的参数 (简单即区别与后面的复杂情况)

* 创建 `launch.json` : 在整个 Project 的根目录下, 点击 `run->Add Configuration...` 此时会自动在 Project 根目录下创建一个 `.vscode` 文件夹, 其中包含一个 `launch.json` 文件

* 在 ` "configurations"` 中加入 `"args"`，具体实例如下:

  ```json
  {
      // Use IntelliSense to learn about possible attributes.
      // Hover to view descriptions of existing attributes.
      // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
      "version": "0.2.0",
      "configurations": [
          {
              "name": "Python: Current File",
              "type": "python",
              "request": "launch",
              "console": "integratedTerminal",
              "justMyCode": true,
              "module": "tools.run_rl",
              "args": [
                  "configs/mbpo/mbpo_mani_skill_state_1M_train.py",
                  "--seed=0",
                  "--cfg-options", "env_cfg.env_name=OpenCabinetDrawer_1000_link_0-v0",
                  "--gpu-ids=6", 
                  "--clean-up"
              ]
          }
      ]
  }
  ```
  
  

* 按下 `F5` 即可, 或者点击右侧 `Run and Debug`, 再点击左上角三角形, 不要去点击右上角的 `Debug Python File`!

## 带参数 -m

有时原始命令可能会带有参数 `-m`, 就不能直接用上面的方法. 据网络查阅, `-m` 命令上是引入一个模块, 所以在 ` "configurations"` 中加入 `"module"`. 例如, 如果原始命令为 `python -m tools.run_rl configs/mbpo/mbpo_mani_skill_state_1M_train.py --seed=0 --cfg-options "env_cfg.env_name=OpenCabinetDrawer_1000_link_0-v0" --gpu-ids=6 --clean-up`, `launch.json` 文件设置如下:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "console": "integratedTerminal",
            "justMyCode": true,
            "module": "tools.run_rl",
            "args": [
                "configs/mbpo/mbpo_mani_skill_state_1M_train.py",
                "--seed=0",
                "--cfg-options", "env_cfg.env_name=OpenCabinetDrawer_1000_link_0-v0",
                "--gpu-ids=6", 
                "--clean-up"
            ]
        }
    ]
}
```

从这里也可以看出