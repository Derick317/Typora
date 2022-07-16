# My Bugs When Coding Python

## `for` Loop

Original code:

```python
for j in not_done:
    traj, info = self.workers[j].get()
    for key in traj:
        trajectories[key][j].append(traj[key])
        for key in info:
            infos[key][j].append(info[key])
            if traj['episode_dones']:
                not_done.remove(j)
```

Despription:

Initially, `not_done = [0, 1]`. At some time, `traj['episode_dones']` would become `True` for both `j = 0` and `j = 1`. I have thought that after executing `not_done.remove(0)`, `not_done` would had become `[1]`, then `j` would had become `1` and another iteration begun.