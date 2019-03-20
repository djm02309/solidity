# Solidity Extension (Sol+)

This extension of Solidity allows additional customized keywords.
The very first keyword is `NonFallBackON`.
For instace, following codelet will check if any function call in  the call chain from `f()` is made to a fallback function; if so, it will stop the execution, rewind the callchain.

    ...
    NonFallBackON;
    f();
    NonFallBackOFF;
    ...

We assume the new keywords will generate NONFALLBACKON, NONFALLBACKOFF instruction for the EVM byte code (for 0x25- for the binary code,) which are again newly introduced.

Example)
A Sol+ source file (including "NonFallBack" keyword)
<pre> <code>
pragma solidity ^0.4.0;
contract Caller{
        function get() public view returns (uint retVal){
                return storedData;
        }
        uint storedData;
        function set (uint x) public payable {
                NonFallBackON;
                storedData = x;
                NonFallBackOFF;
        }
}

contract Callee{
        address public sender;
        uint public recvEther;

        function() external payable{
                sender = msg.sender;
                recvEther += msg.value;
 }
}

</code> </pre>

The result byte code will be as follows;

(1) Caller's code (NONFALLBACKON, NONFALLBACKOFF in the middle)

<pre> <code>
PUSH1 0x80 PUSH1 0x40 MSTORE CALLVALUE DUP1 ISZERO PUSH2 0x10 JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST POP PUSH1 0xE2 DUP1 PUSH2 0x1F PUSH1 0x0 CODECOPY PUSH1 0x0 RETURN INVALID PUSH1 0x80 PUSH1 0x40 MSTORE PUSH1 0x4 CALLDATASIZE LT PUSH1 0x49 JUMPI PUSH1 0x0 CALLDATALOAD PUSH29 0x100000000000000000000000000000000000000000000000000000000 SWAP1 DIV PUSH4 0xFFFFFFFF AND DUP1 PUSH4 0x60FE47B1 EQ PUSH1 0x4E JUMPI DUP1 PUSH4 0x6D4CE63C EQ PUSH1 0x79 JUMPI JUMPDEST PUSH1 0x0 DUP1 REVERT JUMPDEST PUSH1 0x77 PUSH1 0x4 DUP1 CALLDATASIZE SUB PUSH1 0x20 DUP2 LT ISZERO PUSH1 0x62 JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST DUP2 ADD SWAP1 DUP1 DUP1 CALLDATALOAD SWAP1 PUSH1 0x20 ADD SWAP1 SWAP3 SWAP2 SWAP1 POP POP POP PUSH1 0xA1 JUMP JUMPDEST STOP JUMPDEST CALLVALUE DUP1 ISZERO PUSH1 0x84 JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST POP PUSH1 0x8B PUSH1 0xAD JUMP JUMPDEST PUSH1 0x40 MLOAD DUP1 DUP3 DUP2 MSTORE PUSH1 0x20 ADD SWAP2 POP POP PUSH1 0x40 MLOAD DUP1 SWAP2 SUB SWAP1 RETURN JUMPDEST NONFALLBACKON DUP1 PUSH1 0x0 DUP2 SWAP1 SSTORE POP NONFALLBACKOFF POP JUMP JUMPDEST PUSH1 0x0 DUP1 SLOAD SWAP1 POP SWAP1 JUMP INVALID LOG1 PUSH6 0x627A7A723058 KECCAK256 STATICCALL DUP11 0xea STOP 0xb1 PUSH24 0xB9BDF2F82B66BB03D1BAD3BB7F91CE19EA2E4D71DE5213FA SGT 0xd8 STOP 0x29
</code> </pre>

(2) Callee's code (STARTFALLBACK and ENDFALLBACK in the middle)

<pre> <code>
PUSH1 0x80 PUSH1 0x40 MSTORE CALLVALUE DUP1 ISZERO PUSH2 0x10 JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST POP PUSH2 0x179 DUP1 PUSH2 0x20 PUSH1 0x0 CODECOPY PUSH1 0x0 RETURN INVALID PUSH1 0x80 PUSH1 0x40 MSTORE PUSH1 0x4 CALLDATASIZE LT PUSH2 0x4C JUMPI PUSH1 0x0 CALLDATALOAD PUSH29 0x100000000000000000000000000000000000000000000000000000000 SWAP1 DIV PUSH4 0xFFFFFFFF AND DUP1 PUSH4 0x5C890712 EQ PUSH2 0xA0 JUMPI DUP1 PUSH4 0x67E404CE EQ PUSH2 0xCB JUMPI JUMPDEST STARTFALLBACK CALLER PUSH1 0x0 DUP1 PUSH2 0x100 EXP DUP2 SLOAD DUP2 PUSH20 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF MUL NOT AND SWAP1 DUP4 PUSH20 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF AND MUL OR SWAP1 SSTORE POP CALLVALUE PUSH1 0x1 PUSH1 0x0 DUP3 DUP3 SLOAD ADD SWAP3 POP POP DUP2 SWAP1 SSTORE POP ENDFALLBACK STOP JUMPDEST CALLVALUE DUP1 ISZERO PUSH2 0xAC JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST POP PUSH2 0xB5 PUSH2 0x122 JUMP JUMPDEST PUSH1 0x40 MLOAD DUP1 DUP3 DUP2 MSTORE PUSH1 0x20 ADD SWAP2 POP POP PUSH1 0x40 MLOAD DUP1 SWAP2 SUB SWAP1 RETURN JUMPDEST CALLVALUE DUP1 ISZERO PUSH2 0xD7 JUMPI PUSH1 0x0 DUP1 REVERT JUMPDEST POP PUSH2 0xE0 PUSH2 0x128 JUMP JUMPDEST PUSH1 0x40 MLOAD DUP1 DUP3 PUSH20 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF AND PUSH20 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF AND DUP2 MSTORE PUSH1 0x20 ADD SWAP2 POP POP PUSH1 0x40 MLOAD DUP1 SWAP2 SUB SWAP1 RETURN JUMPDEST PUSH1 0x1 SLOAD DUP2 JUMP JUMPDEST PUSH1 0x0 DUP1 SWAP1 SLOAD SWAP1 PUSH2 0x100 EXP SWAP1 DIV PUSH20 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF AND DUP2 JUMP INVALID LOG1 PUSH6 0x627A7A723058 KECCAK256 PUSH5 0xA63B43BE76 POP CALLCODE PUSH14 0x5AF939DC42C82B98640C95514972 DUP10 0x4f 0xe6 DUP15 0x48 LOG4 ADD MULMOD 0xaa STOP 0x29
</code> </pre>

Modified C/C++ source files of Solidity repository are;

1. libevmasm/Instruction.cpp
2. libevmasm/Instruction.h
3. libsolidity/ast/AST_accept.h
4. libsolidity/ast/ASTForward.h
5. libsolidity/ast/AST.h
6. libsolidity/ast/ASTJsonConverter.cpp
7. libsolidity/ast/ASTJsonConverter.h
8. libsolidity/ast/ASTPrinter.cpp
9. libsolidity/ast/ASTPrinter.h
10. libsolidity/ast/ASTVisitor.h
11. libsolidity/codegen/CompilerContext.cpp, CompilerContext.h
12. libsolidity/codegen/ContractCompiler.cpp, ContractCompiler.h
13. libsolidity/parsing/Parser.cpp
14. libsolidity/parsing/Parser.h 
15. libsolidity/parsing/Token.h



For more details, please contact us. (fcmonoid@gmail.com)
