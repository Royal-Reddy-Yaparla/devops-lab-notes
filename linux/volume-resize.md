lsblk

sudo growpart /dev/nvme0n1 4


sudo lvextend -L +20G /dev/RootVG/rootVol
sudo lvextend -L +10G /dev/RootVG/varVol


df -hT


sudo xfs_growfs /
sudo xfs_growfs /home
