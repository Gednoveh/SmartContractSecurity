Delegatecall
==

Delegatecall特点
--

1.部署合约A  

        // SPDX-License-Identifier: MIT
        pragma solidity ^0.8.0;

        contract A {
            address public c;
            address public a;

            function test() public returns (address b){
                b = address(this);
                a = b;

            }
        }

2.部署合约B  

        // SPDX-License-Identifier: MIT
        pragma solidity ^0.8.0;

        contract B {
            address public a;
            address public c;

            address Aaddress = [contract A address];

            function testDelegatecall() public{
                Aaddress.delegatecall(abi.encodeWithSignature("test()"));
            }
        }

3.使用A合约中的<code>test()</code>方法，查看a和c，a被修改为合约A地址  

![image](https://user-images.githubusercontent.com/35074461/197927859-cb98420f-5f78-4143-b045-fa459671d22b.png)

4.使用B合约中的<code>testDelegatecall()</code>方法，外部调用<code>test()</code>方法.查看a和c，c被修改为合约C地址  

![image](https://user-images.githubusercontent.com/35074461/197928130-d69db6b8-46e1-41bf-a448-31a240b6b7f1.png)

