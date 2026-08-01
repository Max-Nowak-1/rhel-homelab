## Screenshots

### Figure 1 – Physical Volume and Volume Group

![Physical Volume and Volume Group](images/lab02-pvs-vgs.png)

The output of `pvs` and `vgs` confirms that the Physical Volume was successfully initialized and added to the Volume Group.

---

### Figure 2 – Logical Volumes

![Logical Volumes](images/lab02-lvs-vgdisplay.png)

The output of `lvs` and `vgdisplay` confirms that both Logical Volumes were successfully created within `vg100`.

## Screenshots

### Figure 1 – Expanded Volume Group and Logical Volume

![Expanded Volume Group and Logical Volume](images/lab03-vg-lv.png)

The output of `pvs`, `vgs`, and `lvs` confirms that the new Physical Volume (`/dev/sdd1`) was successfully added to `vg100` and that `lvol0` was expanded from 160 MiB to approximately 304 MiB.