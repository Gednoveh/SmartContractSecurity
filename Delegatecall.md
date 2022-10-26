Delegatecall
==

Delegatecall特点用例
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

3.使用合约B中的<code>test()</code>方法，查看a和c，a被修改为合约A地址  

![image](https://user-images.githubusercontent.com/35074461/197927859-cb98420f-5f78-4143-b045-fa459671d22b.png)

4.使用合约B中的<code>testDelegatecall()</code>方法，外部调用<code>test()</code>方法。查看a和c，c被修改为合约C地址  

![image](https://user-images.githubusercontent.com/35074461/197928130-d69db6b8-46e1-41bf-a448-31a240b6b7f1.png)

5.部署合约C  

        // SPDX-License-Identifier: MIT
        pragma solidity ^0.8.0;

        contract C {
            address public a;
            address public c;

            address Aaddress = [contract A address];

            function testcall() public{
                Aaddress.call(abi.encodeWithSignature("test()"));
            }
        }
6.使用合约C中的<code>testcall()</code>方法，外部调用<code>test()</code>方法。查看a和c，发现两个变量都没有发生修改  

![image](https://user-images.githubusercontent.com/35074461/197959815-7d916679-7854-496a-a3ec-3e9656e20ccf.png)

特点总结
--

1.使用<code>delegatecall</code>函数进行外部调用涉及到 storage 变量的修改时，变量的修改并不是根据变量名，而是根据变量的存储插槽位置。  

      A 合约中 address c 存储在 slot0 中，address a 存储在 slot1 中，而在 B 合约中 address a 存储在 slot0 中，address c 存储在 slot1 中。  

      当我们通过调用 B 合约中的 delegatecall 函数调用 A 合约中的 test 函数时，test 函数修改的是 A 合约中 slot1 这个插槽，所以代码运行的结果是 B 合约中的 address c 被修改了。
2.使用<code>call</code>函数进行外部调用时，被调用方法的运行环境是被调用者的运行环境，不会对调用者产生影响  

3.使用<code>delegatecall</code>函数进行外部调用，被调用方法的运行环境是调用者的运行环境，会影响调用者

安全建议
--


1.在使用 delegatecall 时应注意被调用合约的地址不能是可控的  

2.在较为复杂的合约环境下需要注意变量的声明顺序以及存储位置。因为使用 delegatecall 进行外部调时会根据被调用合约的数据结构来用修改本合约相应 slot 中存储的数据，在数据结构发生变化时这可能会造成非预期的变量覆盖。  


