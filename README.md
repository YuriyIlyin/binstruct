[![Go Report Card](https://goreportcard.com/badge/github.com/ghostiam/binstruct)](https://goreportcard.com/report/github.com/ghostiam/binstruct) [![Build Status](https://travis-ci.com/ghostiam/binstruct.svg?branch=master)](https://travis-ci.com/ghostiam/binstruct) [![CodeCov](https://codecov.io/gh/ghostiam/binstruct/branch/master/graph/badge.svg)](https://codecov.io/gh/ghostiam/binstruct) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/ghostiam/binstruct/blob/master/LICENSE)

# binstruct
Golang binary decoder to structure

# Examples

[ZIP decoder](examples/zip) \
[PNG decoder](examples/png)

# Use

## For struct

### From file or other io.ReadSeeker:
```go
file, err := os.Open("sample.png")
if err != nil {
    log.Fatal(err)
}

var png PNG
decoder := binstruct.NewDecoder(file, binary.BigEndian)
decoder.SetDebug(true) // you can enable the output of bytes read for debugging
err = decoder.Decode(&png)
if err != nil {
    log.Fatal(err)
}

spew.Dump(png)
```

### From bytes

```go
data := []byte{
    0x00, 0x01,
    0x00, 0x02,
    0x00, 0x03,
    0x00, 0x04,
}

type dataStruct struct {
    Arr []int16 `bin:"len:4"`
}

var actual dataStruct
err := UnmarshalBE(data, &actual) // UnmarshalLE() or Unmarshal()
if err != nil {
    log.Fatal(err)
}
```

## or just use reader without mapping data into the structure

You can not use the functionality for mapping data into the structure, you can use the interface to get data from the stream (io.ReadSeeker)

[reader.go](reader.go)
```go
type Reader interface {
	io.ReadSeeker

	Peek(n int) ([]byte, error)

	ReadBytes(n int) (an int, b []byte, err error)
	ReadAll() ([]byte, error)

	ReadByte() (byte, error)
	ReadBool() (bool, error)

	ReadUint8() (uint8, error)
	ReadUint16() (uint16, error)
	ReadUint32() (uint32, error)
	ReadUint64() (uint64, error)

	ReadInt8() (int8, error)
	ReadInt16() (int16, error)
	ReadInt32() (int32, error)
	ReadInt64() (int64, error)

	ReadFloat32() (float32, error)
	ReadFloat64() (float64, error)

	Unmarshaler
}
```

# Decode to fields

```go
type test struct {
	// Read 1 byte
	Field bool
	Field byte
	Field [1]byte
	Field int8
	Field uint8

	// Read 2 bytes
	Field int16
	Field uint16
	Field [2]byte

	// Read 4 bytes
	Field int32
	Field uint32
	Field [4]byte

	// Read 8 bytes
	Field int64
	Field uint64
	Field [8]byte

	// You can override length
	Field int64 `bin:"len:2"`

	// Fields of type int, uint and string are not read automatically 
	// because the size is not known, you need to set it manually
	Field int    `bin:"len:2"`
	Field uint   `bin:"len:4"`
	Field string `bin:"len:42"`
	
	// Can read arrays and slices
	Array [2]int32              // read 8 bytes (4+4byte for 2 int32)
	Slice []int32 `bin:"len:2"` // read 8 bytes (4+4byte for 2 int32)
	
	// Also two-dimensional slices work (binstruct_test.go:209 Test_SliceOfSlice)
	Slice2D [][]int32 `bin:"len:2,[len:2]"`
	// and even three-dimensional slices (binstruct_test.go:231 Test_SliceOfSliceOfSlice)
	Slice3D [][][]int32 `bin:"len:2,[len:2,[len:2]]"`
	
	// Structures and embedding are also supported.
	Struct struct {
		...
	}
	OtherStruct Other
	Other // embedding
}

type Other struct {
	...
}
```

# Tags

```go
type test struct {
	IgnoredField []byte `bin:"-"`          // ignore field
	CallMethod   []byte `bin:"MethodName"` // Call method "MethodName"
	ReadLength   []byte `bin:"len:42"`     // read 42 bytes
	
	// Offsets test binstruct_test.go:9
	Offset      byte `bin:"offset:42"`      // move to 42 bytes from current position and read byte
	OffsetStart byte `bin:"offsetStart:42"` // move to 42 bytes from start position and read byte
	OffsetEnd   byte `bin:"offsetEnd:-42"`  // move to -42 bytes from end position and read byte
	OffsetStart byte `bin:"offsetStart:42, offset:10"` // also worked and equally `offsetStart:52`

	// Calculations supported +,-,/,* and are performed from left to right that is 2+2*2=8 not 6!!!
	CalcTagValue []byte `bin:"len:10+5+2+3"` // equally len:20
	
	// You can refer to another field to get the value.
	DataLength              int    // actual length
	ValueFromOtherField     string `bin:"len:DataLength"`
	CalcValueFromOtherField string `bin:"len:DataLength+10"` // also work calculations
} 

// Method can be:
func (*test) MethodName(r binstruct.Reader) (error) {}
// or
func (*test) MethodName(r binstruct.Reader) (FieldType, error) {}
```

See the tests and sample files for more information.

# License

MIT License