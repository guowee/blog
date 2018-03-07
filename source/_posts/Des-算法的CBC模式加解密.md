---
title: Des 算法的CBC模式加解密
date: 2018-03-07 15:27:50
tags: 
---

# 加密
1. 首先将数据按照8字节一组进行分组得到D1D2D3...Dn.
2. 第一组数据D1与初始向量I(初始向量I为全零)异或后的结果进行Des加密运算得到第一组密文C1.
3. 第二组数据D2与第一组的加密结果C1异或后的结果进行Des加密运算，得到第二组密文C2.
4. 之后的数据以此类推，得到Cn
5. 按顺序连接C1，C2，C3，...,Cn,即为加密结果.
```
 public byte[] desCBCEncrypt(byte keyIndex, byte[] initvector, byte[] datain) throws CustomException {

        if (datain.length % 8 != 0) {
            throw new CustomException(ErrorCode.COMMON_ERR_PARAM);
        }

        int offset = 0;
        byte[] result = new byte[datain.length];
        byte[] ret = initvector;
        try {
            while (offset < datain.length) {

                byte[] temp = new byte[8];
                System.arraycopy(datain, offset, temp, 0, 8);

                byte[] _temp = xor(temp, ret, 8);

                ret = ped.calcDes(keyIndex, _temp, EPedDesMode.ENCRYPT);

                System.arraycopy(ret, 0, result, offset, 8);
                offset += 8;
            }
            return result;
        } catch (PedDevException e) {
            e.printStackTrace();
            throw new CustomException(ErrorCode.PED_ERR - e.getErrCode());
        }
    }
```
# 解密
1. 首先将数据按照8个字节一组进行分组得到C1，C2，C3，...,Cn.
2. 将第一组数据进行解密后与初始向量I进行异或得到第一组明文D1.(一定是先解密再异或)
3. 将第二组数据C2进行解密后与第一组密文数据C1进行异或得到第二组明文D2.
4. 之后的数据以此类推，得到Dn.
5. 按照顺序连接D1，D2，D3，...,Dn,即为解密结果.（解密结果并不一定是原来的加密数据，可能还含有补位，一定要把补位去掉才是原数据）
```
 public byte[] desCBCDecrypt(byte keyIndex, byte[] initvector, byte[] datain) throws CustomException {

        if (datain.length % 8 != 0) {
            throw new CustomException(ErrorCode.COMMON_ERR_PARAM);
        }
        
        int offset = 0;
        byte[] result = new byte[datain.length];
        byte[] ret = initvector;
        try {
            while (offset < datain.length) {
                byte[] temp = new byte[8];
                System.arraycopy(datain, offset, temp, 0, 8);

                byte[] _temp = ped.calcDes(keyIndex, temp, EPedDesMode.DECRYPT);
                ret = xor(_temp, ret, 8);

                System.arraycopy(ret, 0, result, offset, 8);
                ret = temp;
                offset += 8;
            }
            
            return result;
        } catch (PedDevException e) {
            e.printStackTrace();
            throw new CustomException(ErrorCode.PED_ERR - e.getErrCode());
        }

    }
	
	private byte[] xor(byte[] a, byte[] b, int len) {
        if (a == null || b == null) {
            return null;
        }

        if (len > a.length || len > b.length) {
            return null;
        }

        byte[] result = new byte[len];
        for (int i = 0; i < len; i++) {
            result[i] = (byte) (a[i] ^ b[i]);
        }
        return result;
    }
```










