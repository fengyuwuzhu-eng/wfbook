# RBD命令

## 检查Pool

Ceph 启动后默认创建 rbd 这个 pool，使用rbd命令默认使用它，也可以创建新的 pool。

```bash
rados lspools
ceph osd pool create rbd_pool 1024
```

## 创建 Image

使用 rbd 命令创建 image，创建后发现 rbd 这 个pool 会多一个`rbd_directory`的 object。

```bash
rbd create test_image --size 1024
rbd ls
rbd --image test_image info
rados -p rbd ls
```

## 修改Image大小

增加Image大小可以直接使用`resize`子命令，如果缩小就需要添加`--allow-shrink`参数保证安全。

```bash
rbd --image test_image resize --size 2000
rbd --image test_image resize --size 1000 --allow-shrink
```

## 使用Image

通过`map`子命令可以把镜像映射成本地块设备，然后就可以格式化和`mount`了。

```bash
rbd map test_image
rbd showmapped
mkfs.ext4 /dev/rbd0
mount /dev/rbd0 /mnt/
```

## 移除Image

```bash
umount /dev/rbd0
rbd unmap /dev/rbd0
rbd showmapped
```

## 删除Image

删除和Linux类似使用`rm`命令即可。

```bash
rbd --image test_image rm
```

