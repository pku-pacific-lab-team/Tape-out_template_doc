# 顶层 LVS/DRC/Antenna 物理验证

## 1. 验证流程

1. 确保已经完成了电源连接线、Seal Ring 的合并（`make merge`），并设置好 `config/user_define.tcl` 中的 `project` 变量。
2. 查看 LVS 验证结果：`make lvs` 后查看 `pv/reports/lvs.summary`。
3. 查看添加 Dummy 前 DRC 验证结果：`make drc` 后查看 `pv/reports/<project_name>_wo_dummy.drc.summary`。
4. 查看添加 Dummy 前天线验证结果：`make ant` 后查看 `pv/reports/<project_name>_wo_dummy.antenna.summary`。
5. 查看添加 Dummy 前 PAD 验证结果：`make wb` 后查看 `pv/reports/<project_name>_wo_dummy.wirebond.summary`。
6. 一键运行添加 Dummy 前所有验证：`make all`。
7. 添加 Dummy：`make dummy`。
8. 查看添加 Dummy 后 DRC 验证结果：`make drc DM=1` 后查看 `pv/reports/<project_name>.drc.summary`。
9. 查看添加 Dummy 后天线验证结果：`make ant DM=1` 后查看 `pv/reports/<project_name>.antenna.summary`。
10. 查看添加 Dummy 后 PAD 验证结果：`make wb DM=1` 后查看 `pv/reports/<project_name>.wirebond.summary`。
11. 一键运行添加 Dummy 后所有验证：`make all DM=1`。

## 2. Final Check

!!! Warning "Final Check"
    在完成 Dummy Fill 之后，确保 DRC、Antenna、PAD 所有验证**均通过**，该 GDS 可以作为最终交付 TSMC 的文件。
