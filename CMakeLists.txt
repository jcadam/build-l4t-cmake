cmake_minimum_required(VERSION 3.0.2)
project(l4t)

set(TOP ${CMAKE_BINARY_DIR})

file(MAKE_DIRECTORY dl prebuilts src doc)

function(download base_url filename path sha1sum)
  set(url ${base_url}/${filename})
  set(fullpath ${TOP}/${path}/${filename})
  file(DOWNLOAD ${url} ${fullpath} EXPECTED_HASH SHA1=${sha1sum} SHOW_PROGRESS)
endfunction(download)

function(l4t_download_prebuilt filename sha1sum)
  message(STATUS "Downloading prebuilt: ${filename}")
  download(http://developer.download.nvidia.com/embedded/L4T/r21_Release_v4.0
    ${filename} dl ${sha1sum})
endfunction(l4t_download_prebuilt)

function(l4t_download_src filename sha1sum)
  message(STATUS "Downloading source: ${filename}")
  download(http://developer.download.nvidia.com/embedded/L4T/r21_Release_v4.0/source/
    ${filename} dl ${sha1sum})
endfunction(l4t_download_src)

function(l4t_download_doc filename sha1sum)
  message(STATUS "Downloading document: ${filename}")
  download(http://developer.download.nvidia.com/embedded/L4T/r21_Release_v4.0
    ${filename} doc ${sha1sum})
endfunction(l4t_download_doc)

function(extract package path)
  execute_process(
    COMMAND ${CMAKE_COMMAND} -E tar xf ${package}
    WORKING_DIRECTORY ${path}
    )
endfunction(extract)

function(extract_src package_name)
  message(STATUS "Extracting source: ${package_name}")
  set(package ${TOP}/dl/${package_name})
  set(path ${TOP}/src/)
  extract(${package} ${path})
endfunction(extract_src)


l4t_download_prebuilt(Tegra124_Linux_R21.4.0_armhf.tbz2
  15da784ef2b6c6f4e5f1921257e83ee922cfd8bb)
l4t_download_prebuilt(Tegra_Linux_Sample-Root-Filesystem_R21.4.0_armhf.tbz2
  edaabb28d70c38c185e379c6277c7d8e6a04f7e6)

l4t_download_doc(Tegra_Linux_Driver_Package_Release_Notes_R21.4.pdf
  1d092130be58056054ad1004158fda8dbee3737d)
l4t_download_doc(Tegra_Linux_Driver_Package_Documents_R21.4.tar
  045f89c169528a5def14c9482a6fbb68e785f8e7)
l4t_download_doc(Tegra_Linux_Driver_Package_SW_Features_R21.4.pdf
  8569d06093031e92aaf263875731729b1891b30e)

l4t_download_src(kernel_src.tbz2
  7e0417c844a5ab0113acda6d88f071481fbd05a5)
#l4t_download_src(ubuntu_14.04-LATEST-LTS_src.tbz2
#  556dcf09d67d33aa578d402e67e145bfe9dd60b2)
l4t_download_src(gstegl_src.tbz2
  0916b19f1c6ffb2ac791e2fd54b87589267b5ab7)
l4t_download_src(u-boot_src.tbz2
  f592ddf3304e77e71aff2c133ccd315a0d84bdc0)
l4t_download_src(gstomx1_src.tbz2
  704cf0d8e11a07441b743100f161b6990d52e31e)
l4t_download_src(gstomx_src.tbz2
  7b87bc2c57d7f77572d4cd34c76f4ff16599afee)
l4t_download_src(gstjpeg_src.tbz2
  4744fae2189cd0cd202e9cfdef0b1dcf0571add1)

file(MAKE_DIRECTORY doc/html)
extract(${TOP}/doc/Tegra_Linux_Driver_Package_Documents_R21.4.tar
  ${TOP}/doc/html)

extract_src(kernel_src.tbz2)
extract_src(u-boot_src.tbz2)

extract(${TOP}/dl/Tegra124_Linux_R21.4.0_armhf.tbz2 ${TOP}/prebuilts/)

file(WRITE ${TOP}/build_sample_and_flash.sh
  "tar xf ${TOP}/dl/Tegra_Linux_Sample-Root-Filesystem_R21.4.0_armhf.tbz2 \
       -C ${TOP}/prebuilts/Linux_for_Tegra/rootfs && \
   ${TOP}/prebuilts/Linux_for_Tegra/apply_binaries.sh && \
   cd ${TOP}/prebuilts/Linux_for_Tegra/ && \
   ./flash.sh jetson-tk1 mmcblk0p1 && \
   cd -")
