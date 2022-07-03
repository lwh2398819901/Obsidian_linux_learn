## b
```toc
```
## blkid
<iframe 
 height=500
 width=900  
src="https://linux.cn/article-4734-1.html"　
>
</iframe>
```bash
root@uos-PC /h/u/Desktop# blkid
/dev/nvme0n1p1: LABEL_FATBOOT="EFI" LABEL="EFI" UUID="7CA3-3E11" TYPE="vfat" PARTUUID="81af3950-b94d-4b11-a2c8-41ed26ae454e"
/dev/nvme0n1p2: LABEL="Boot" UUID="ec336ab8-b333-41cc-aad0-211769c69a44" TYPE="ext4" PARTUUID="ab3224d6-5cd2-414a-9ca1-0a02ed63568b"
/dev/nvme0n1p3: LABEL="Roota" UUID="fc0f5306-16f1-4a2c-b9a2-8434f8d72bc9" TYPE="ext4" PARTUUID="142f0f48-cf2b-445f-9a1c-d75d7321259d"
/dev/nvme0n1p4: LABEL="Rootb" UUID="3512cef6-368f-49af-ae3e-3739f9ef4c66" TYPE="ext4" PARTUUID="f7201b10-02b0-48f0-b787-385a90b28b35"
/dev/nvme0n1p5: LABEL="_dde_data" UUID="934bf91a-a943-417a-b679-b33b8cdabc55" TYPE="ext4" PARTUUID="d1235fc8-1612-44c5-9cde-5fde31e76db1"
/dev/nvme0n1p6: LABEL="Backup" UUID="2dcf9896-698e-4374-99c9-fcdfd6856caa" TYPE="ext4" PARTUUID="bc6e1163-df45-4ff7-8749-188d777b3488"
/dev/nvme0n1p7: LABEL="SWAP" UUID="322859f6-63b9-469d-9efb-b3b125d7aa4e" TYPE="swap" PARTUUID="f3a9b123-e501-462e-862f-8afb6a4e742e"
/dev/sda2: UUID="2E50C0F250C0C237" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="29984ad3-2bba-4f0a-8820-c99671340970"
/dev/sda3: LABEL="ext4" UUID="4723d1da-fbf7-46ba-9715-31a5df2eca18" TYPE="ext4" PARTUUID="fab1bb52-74a5-432e-9cb7-f3ee09d1028a"
/dev/loop0: TYPE="squashfs"
/dev/loop1: TYPE="squashfs"
/dev/loop2: TYPE="squashfs"
/dev/loop3: TYPE="squashfs"
/dev/loop4: TYPE="squashfs"
/dev/nvme0n1: PTUUID="bc629b89-57ac-4145-aba3-39a0d75a7160" PTTYPE="gpt"
/dev/sda1: PARTLABEL="Microsoft reserved partition" PARTUUID="2ded83c1-b239-49