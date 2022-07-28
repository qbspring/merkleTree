# MerkleTree
Aiming to full compatibility with OpenZeppelin's implementation in Solidity.

#### Install
```
go get github.com/qbspring/merkletree
```

#### Example Usage
Below is an example that makes use of the entire API - its quite small.

```go
package main

import (
	"encoding/hex"
	"fmt"
	"github.com/qbspring/merkletree"
	"golang.org/x/crypto/sha3"
)

func main() {
	var datas = []string{
		"0xa00ac0899db0c8bd8620d3fbce7c668a375178ad46267e64a6caa7324c932a49",
		"0x08049b0d7aac70b8a9b7881c90ba206761e898241528041c43c9d4524d82e03b",
	}

	leaves := [][]byte{}
	for _, data := range datas {
		leaves = append(leaves, generateLeaves(data))
	}

	tree, err := merkletree.New(leaves, &merkletree.Options{
		HashObj:   sha3.NewLegacyKeccak256(),
		SortPairs: true,
	})
	if err != nil {
		panic(err)
	}

	root := tree.Root()
	leave := generateLeaves(datas[1])
	proof := tree.GetProof(leave)
	result := merkletree.VerifySorted(root, leave, proof, sha3.NewLegacyKeccak256())
	if result {
		fmt.Println("merkle tree root:", tree.HexRoot())
		fmt.Println("index 1 proof:", tree.GetHexProof(leave))
		fmt.Println("verify success")
	} else {
		fmt.Println("verify fail")
	}

}

func generateLeaves(data string) []byte {
	return keccak256(hexToByte(data))
}

func hexToByte(hexString string) []byte {
	data, err := hex.DecodeString(hexString[2:])
	if err != nil {
		panic(err)
	}
	return data
}

func keccak256(data []byte) []byte {
	h := sha3.NewLegacyKeccak256()
	h.Write(data)
	return h.Sum(nil)
}

```