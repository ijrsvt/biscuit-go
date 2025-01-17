# Biscuit samples and expected results

root secret key: 12aca40167fbdd1a11037e9fd440e3d510d9d9dea70a6646aa4aaf84d718d75a
root public key: acdd6d5b53bfee478bf689f8e012fe7988bf755e3d7c5152947abc149bc20189

------------------------------

## basic token: test1_basic.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

authorizer code:
```
resource("file1");
```

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:9d3e984bd0447eea9f31a56df51ba606160c66102063dd29410a2c85601a2139ce0cd212daf755ed0b8fe1f0e9388a89074b009b7169499e51df83c308e8d20b)",
    "revocation_id(1, hex:5cade9fd3690b72bf90c29c529cb5b1bb50832554ba525b15c5d3f7c994814af522c5a68d61a950bc5f98d9ff4e3e20ffecef65ddaa2858251768ec999ed8b06)",
    "right(\"file1\", \"read\")",
    "right(\"file1\", \"write\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## different root key: test2_different_root_key.bc
### token

authority:
symbols: ["file1", "read"]

```
right("file1", "read");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## invalid signature format: test3_invalid_signature_format.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(InvalidSignatureSize(16)))`


------------------------------

## random block: test4_random_block.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## invalid signature: test5_invalid_signature.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## reordered blocks: test6_reordered_blocks.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: ["check2"]

```
check if resource("file1");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## scoped rules: test7_scoped_rules.bc
### token

authority:
symbols: ["user_id", "alice", "owner", "file1"]

```
user_id("alice");
owner("alice", "file1");
```

1:
symbols: ["0", "read", "1", "check1"]

```
right($0, "read") <- resource($0), user_id($1), owner($1, $0);
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: ["file2"]

```
owner("alice", "file2");
```

### validation

authorizer code:
```
resource("file2");
operation("read");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "owner(\"alice\", \"file1\")",
    "owner(\"alice\", \"file2\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:d2454c600567418982b2787c1fbc4e04d6f59f1576b6613d1cacd30440f673a0c44728457a39fb8085e4152a8195e0bdfbe3a5fdcfafd08b33ad53c3274c6d0c)",
    "revocation_id(1, hex:aad436b9239c4df033f0ad88276981f7738033df4562c0e2ae3da1fa9629c050e00a44e5831520cdb4dba879cfb047cde523ef5fbffc19e5fcd5969177466400)",
    "revocation_id(2, hex:ca46c3c9099242ea594642ea6fa75c47df463b2548f090e0800fc10375d2cd464571c54316cfbee863c01f49ccd72492483d95134090327ea92984202c07d004)",
    "user_id(\"alice\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## scoped checks: test8_scoped_checks.bc
### token

authority:
symbols: ["file1", "read"]

```
right("file1", "read");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: ["file2"]

```
right("file2", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:593c16b2bb2a00c02a9be0504206a142c77917af234ea7b5109b1bad22459fc4e6680ff38c852ca75959f637ebb02479d60d63d47e1514636c34acf3b378c40e)",
    "revocation_id(1, hex:8dddcbff3fd9dfd494b98a9c15225e1064e5c96eaf977e6a06e6581bdea2440c67ea7a88d7d51badf732217351ead40041beda6d4f892518e46b187207bc840c)",
    "revocation_id(2, hex:587e3b1a03c3247db490c246adf0e02e00abda4b2cccb1dbf1adb5ccb5b978d9a9bbf8fcdcc81680e0f9d89e57cb1537a4e71a50e8b1542761b585d9a204f504)",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## expired token: test9_expired_token.bc
### token

authority:
symbols: []

```
```

1:
symbols: ["check1", "file1", "expiration", "date", "time"]

```
check if resource("file1");
check if time($date), $date <= 2018-12-20T00:00:00+00:00;
```

### validation

authorizer code:
```
resource("file1");
operation("read");
time(2020-12-21T09:23:12+00:00);
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "revocation_id(0, hex:16d0a9d7f3d29ee2112d67451c8e4ff07bd5366a6cdb082cf4fcb66e6d15a57a22009ef1018fc4d0f9184edb0900df161807bc6f8287275f32eae6b5b1c57100)",
    "revocation_id(1, hex:0670d948462e0cc248ce45b7ea04cbfb126a7559c8d60b533f7f0a92696900ee4e432780b526462b845d372c9b7b223c43efc22e0441b14b0bc4661e05ebfe03)",
    "time(2020-12-21T09:23:12+00:00)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 1, rule: "check if time($date), $date <= 2018-12-20T00:00:00+00:00" })] }))`


------------------------------

## authorizer scope: test10_authorizer_scope.bc
### token

authority:
symbols: ["file1", "read"]

```
right("file1", "read");
```

1:
symbols: ["file2"]

```
right("file2", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:5e626c4991877dd41d9e506d51a3888454cc764e11622945b24df99ca0bcc7f144d41aea0fb88778e67cf0f8609e47302d11007dc456bcdb98c14a25a6eecc05)",
    "revocation_id(1, hex:1c5896cc25959f456db10fa142164f90e99791313d65025e2058e4f990314f12965a22ca394f448083c64fd29438ff9ad25634320f8907a0587153d905adc108)",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {
    "check if right($0, $1), resource($0), operation($1)",
}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Authorizer(FailedAuthorizerCheck { check_id: 0, rule: "check if right($0, $1), resource($0), operation($1)" })] }))`


------------------------------

## authorizer authority checks: test11_authorizer_authority_caveats.bc
### token

authority:
symbols: ["file1", "read"]

```
right("file1", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:7c0601144e26538ed4870f844a970b2b8bdabab13dd676763956ae9a8e3ec830fbb8a031b92abd4eb66124d9f8d86576a5161cd1499f29539372676fdb740505)",
    "right(\"file1\", \"read\")",
}
  rules: {}
  checks: {
    "check if right($0, $1), resource($0), operation($1)",
}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Authorizer(FailedAuthorizerCheck { check_id: 0, rule: "check if right($0, $1), resource($0), operation($1)" })] }))`


------------------------------

## authority checks: test12_authority_caveats.bc
### token

authority:
symbols: ["check1", "file1"]

```
check if resource("file1");
```

### validation for "file1"

authorizer code:
```
resource("file1");
operation("read");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "revocation_id(0, hex:0d313cc11a09af8844290865c919220aebfb260aa5a1f738c8a8f3df677902e5ea06f408fa316d527926a688764a2c5e06cdecf14bc1ace3e6128323dcb8c801)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`
### validation for "file2"

authorizer code:
```
resource("file2");
operation("read");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:0d313cc11a09af8844290865c919220aebfb260aa5a1f738c8a8f3df677902e5ea06f408fa316d527926a688764a2c5e06cdecf14bc1ace3e6128323dcb8c801)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource(\"file1\")" })] }))`


------------------------------

## block rules: test13_block_rules.bc
### token

authority:
symbols: ["file1", "read", "file2"]

```
right("file1", "read");
right("file2", "read");
```

1:
symbols: ["valid_date", "time", "0", "1", "check1"]

```
valid_date("file1") <- time($0), resource("file1"), $0 <= 2030-12-31T12:59:59+00:00;
valid_date($1) <- time($0), resource($1), $0 <= 1999-12-31T12:59:59+00:00, !["file1"].contains($1);
check if valid_date($0), resource($0);
```

### validation for "file1"

authorizer code:
```
resource("file1");
time(2020-12-21T09:23:12+00:00);
```

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:893ff2daf44325f05849f581de561732094f14223d724202ce2f3d4058cead2ba238e4ef3a6b18f076f155e5e21ec30eded28f98d29979a39eb7f72da128a404)",
    "revocation_id(1, hex:3189fe4ccec73777fcb0a63fb497c4391bc967c1cc02ec409ae19e7e30fd2bfeb2c309e67c615bcae986a0de15a1a21b5623ccdab5afe36c11c539ac7e475202)",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
    "time(2020-12-21T09:23:12+00:00)",
    "valid_date(\"file1\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`
### validation for "file2"

authorizer code:
```
resource("file2");
time(2020-12-21T09:23:12+00:00);
```

authorizer world:
```
World {
  facts: {
    "resource(\"file2\")",
    "revocation_id(0, hex:893ff2daf44325f05849f581de561732094f14223d724202ce2f3d4058cead2ba238e4ef3a6b18f076f155e5e21ec30eded28f98d29979a39eb7f72da128a404)",
    "revocation_id(1, hex:3189fe4ccec73777fcb0a63fb497c4391bc967c1cc02ec409ae19e7e30fd2bfeb2c309e67c615bcae986a0de15a1a21b5623ccdab5afe36c11c539ac7e475202)",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
    "time(2020-12-21T09:23:12+00:00)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if valid_date($0), resource($0)" })] }))`


------------------------------

## regex_constraint: test14_regex_constraint.bc
### token

authority:
symbols: ["resource_match", "0", "file[0-9]+.txt"]

```
check if resource($0), $0.matches("file[0-9]+.txt");
```

### validation for "file1"

authorizer code:
```
resource("file1");
```

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:9752ecf19b270129471b459de5b8fbf6c04ad652d1ebd042f79efd8ceb6d14fd3a92ff5f2ada3996895bc4e9effe2b723b775d28ddcdc2365294a4420b67790f)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource($0), $0.matches(\"file[0-9]+.txt\")" })] }))`
### validation for "file123"

authorizer code:
```
resource("file123.txt");
```

authorizer world:
```
World {
  facts: {
    "resource(\"file123.txt\")",
    "revocation_id(0, hex:9752ecf19b270129471b459de5b8fbf6c04ad652d1ebd042f79efd8ceb6d14fd3a92ff5f2ada3996895bc4e9effe2b723b775d28ddcdc2365294a4420b67790f)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## multi queries checks: test15_multi_queries_caveats.bc
### token

authority:
symbols: ["must_be_present", "hello"]

```
must_be_present("hello");
```

### validation

authorizer code:
```

check if must_be_present($0) or must_be_present($0);
```

authorizer world:
```
World {
  facts: {
    "must_be_present(\"hello\")",
    "revocation_id(0, hex:aa4293d9e62461c2871071a3c40c515427927fa47e7e123e857ba1f41275a87ca53db2183023d09a4ad09cf6c1e70c816a48ab0b532a49c3ebb903cfbc66cf01)",
}
  rules: {}
  checks: {
    "check if must_be_present($0) or must_be_present($0)",
}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## check head name should be independent from fact names: test16_caveat_head_name.bc
### token

authority:
symbols: ["check1", "test", "hello"]

```
check if resource("hello");
```

1:
symbols: []

```
check1("test");
```

### validation

authorizer code:
```
```

authorizer world:
```
World {
  facts: {
    "check1(\"test\")",
    "revocation_id(0, hex:aa8f26e32b6a55fe99decfb0f2c229776cc30360e5b68a5b06e730f1e9a13697f87929592f37b7b58dd00dececd6fa40540a3879f74bd232505f1c419907000c)",
    "revocation_id(1, hex:02766fa2dbb0bd5a2d4d3fc4e0dd9252ec4dc118fe5bc0eafb67fbce0ddf6a86f4db7ecc0b1da14c210b8dcae53fcfc44565edb32ba18bfc9ca9f97258c4db0d)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource(\"hello\")" })] }))`


------------------------------

## test expression syntax and all available operations: test17_expressions.bc
### token

authority:
symbols: ["query", "hello world", "hello", "world", "aaabde", "a*c?.e", "abcD12", "abc", "def"]

```
check if true;
check if !false;
check if false or true;
check if 1 < 2;
check if 2 > 1;
check if 1 <= 2;
check if 1 <= 1;
check if 2 >= 1;
check if 2 >= 2;
check if 3 == 3;
check if 1 + 2 * 3 - 4 / 2 == 5;
check if "hello world".starts_with("hello") && "hello world".ends_with("world");
check if "aaabde".matches("a*c?.e");
check if "abcD12" == "abcD12";
check if 2019-12-04T09:46:41+00:00 < 2020-12-04T09:46:41+00:00;
check if 2020-12-04T09:46:41+00:00 > 2019-12-04T09:46:41+00:00;
check if 2019-12-04T09:46:41+00:00 <= 2020-12-04T09:46:41+00:00;
check if 2020-12-04T09:46:41+00:00 >= 2020-12-04T09:46:41+00:00;
check if 2020-12-04T09:46:41+00:00 >= 2019-12-04T09:46:41+00:00;
check if 2020-12-04T09:46:41+00:00 >= 2020-12-04T09:46:41+00:00;
check if 2020-12-04T09:46:41+00:00 == 2020-12-04T09:46:41+00:00;
check if hex:12ab == hex:12ab;
check if [1, 2].contains(2);
check if [2019-12-04T09:46:41+00:00, 2020-12-04T09:46:41+00:00].contains(2020-12-04T09:46:41+00:00);
check if [false, true].contains(true);
check if ["abc", "def"].contains("abc");
check if [hex:12ab, hex:34de].contains(hex:34de);
```

### validation

authorizer code:
```
```

authorizer world:
```
World {
  facts: {
    "revocation_id(0, hex:39e2c7e2319cc614acf881d06bfd5e344a0e7ed2c4c15e0d068f66467276dead3db6d4aca2cf5b688fc84f13861c7c89c047adde161f962dee18099902da5608)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## invalid block rule with unbound_variables: test18_unbound_variables_in_rule.bc
### token

authority:
symbols: ["check1", "test", "read"]

```
check if operation("read");
```

1:
symbols: ["unbound", "any1", "any2"]

```
operation($unbound, "read") <- operation($any1, $any2);
```

### validation

authorizer code:
```
operation("write");
```

authorizer world:
```
World {
  facts: {
    "operation(\"write\")",
    "revocation_id(0, hex:33756b656cbb74acea3613b37ba27be1c761ebeacfb5143bab0e284febb04f048eda846b1419558f38d08628b141cd1b38a261c6e865d1c8ed65722a839ec803)",
    "revocation_id(1, hex:05b10a427cfb7e4712bf8b56edaba207200a53b68a4e8b79afe935b37791e7ac5bfb89ff6c6f20795a82a8b18d60194b92db55d0a82edd8ce3a744459fe3130b)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(InvalidBlockRule(0, "operation($unbound, \"read\") <- operation($any1, $any2)")))`


------------------------------

## invalid block rule generating an #authority or #ambient symbol with a variable: test19_generating_ambient_from_variables.bc
### token

authority:
symbols: ["check1", "test", "read"]

```
check if operation("read");
```

1:
symbols: ["any"]

```
operation("read") <- operation($any);
```

### validation

authorizer code:
```
operation("write");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "operation(\"write\")",
    "revocation_id(0, hex:f2bb00974734d38dd729b0cf8e6625a63186cc03b43d48b662d7e9f5821f90881359802ebac1fdf3407f15a65c1584363f8ea03f50eb66105df55275415a910c)",
    "revocation_id(1, hex:72f9a076f221f3458db15b373df023245bd0fc811ea28a9f99b79bd908224ea317986692c159a54f3aba1f15ba771c8e3ac6bc998a36e79a08aedbc25f1e200d)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if operation(\"read\")" })] }))`


------------------------------

## sealed token: test20_sealed.bc
### token

authority:
symbols: ["file1", "read", "file2", "write"]

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["check1", "0"]

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

authorizer code:
```
resource("file1");
operation("read");
```

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "revocation_id(0, hex:669be0e6d07eb7a34be1f48921976e70ff9491845f4c983c59bfd0aac449a76c239120f152e1ed10d1c86da73cf7ff6f3bdde0f42e242d0f911e0b938d516c04)",
    "revocation_id(1, hex:05c5f63076fb7ad5d6eef8a486d8a460c8fa8d986e1d8f9a0b28997687b0541fccd42fb974c4ed3032a0f5553f7c8022c4ad734df87e589ca25efcab8552b009)",
    "right(\"file1\", \"read\")",
    "right(\"file1\", \"write\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`

