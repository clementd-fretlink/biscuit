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

```

### validation

verifier code:
```
resource("file1");
```

verifier world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:bd6e89a2b700700cc68e644298685b1283deee82cc119417d03391a652cfa2bd55968f8e6039c48c39daa6a5efe984eb56733e9eb3289d9fb4c310b95c0a3701)",
    "revocation_id(1, hex:588f783d07f5bc0f145c452776494dcbbfed460484e7c06bba82b0f4edfbe2ecac9e97efc420a4344361544a21c6fa1f95dd0aeb4b161c6fbd06b839ffedd80a)",
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

result: `Err(["Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: \"check if resource($0), operation(\\\"read\\\"), right($0, \\\"read\\\")\" })"])`


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

```

### validation

result: `Err(["Format(Signature(InvalidSignature(\"signature error\")))"])`


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

```

### validation

result: `Err(["Format(InvalidSignatureSize(16))"])`


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

```

### validation

result: `Err(["Format(Signature(InvalidSignature(\"signature error\")))"])`


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

```

### validation

result: `Err(["Format(Signature(InvalidSignature(\"signature error\")))"])`


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

```

### validation

result: `Err(["Format(Signature(InvalidSignature(\"signature error\")))"])`


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

```

### validation

verifier code:
```
resource("file2");
operation("read");
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "owner(\"alice\", \"file1\")",
    "owner(\"alice\", \"file2\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:9373e9f4418a9ce4818e5031c7fbd6dadd840c4ea5d9dd8ee088fdbd9f8c9da3a6517ee7fb581ee2a75ac3fe9eb4cc10338e6b877849dc433c7a62d1cd5a9706)",
    "revocation_id(1, hex:6dd0e774476520b616e8b68ee693791e2273d2349adbd1c58ebd987895c5286400b8af081f2cf5d1a565be2d96bb906990c3f4287dbae3dd1ab0fdd2dce31e0a)",
    "revocation_id(2, hex:ce242e513db4cf2dcd8a5cc2cd37313caab903b8f0bd7bfb86c425a9a4af043492325d67ce97ff570667fa2325091caa025d5bb1f68b48fc11bc7b689e78e20e)",
    "user_id(\"alice\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: \"check if resource($0), operation(\\\"read\\\"), right($0, \\\"read\\\")\" })"])`


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

```

### validation

verifier code:
```
resource("file2");
operation("read");
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:8e4fba9d79d7752b74808e9571804778d358f1be3dca8cde638e15683d14a0587e38f39d726a52c93b87c1c6a80e6cffed57761dcc0cd42e2d94819c661b1607)",
    "revocation_id(1, hex:4222d817999f47d1b52dfb4e6457487b69153a8a8b87b9f42160b7210bcfe1d01e8ad752311751fcbf87e20a7a92e5e789b7d09b8539dec7603038f29d2a0a07)",
    "revocation_id(2, hex:05f1a98da4caccc50bda218ead6d535e27cb7a07a1cc7d792ae3ce718a9b01b7066ec5a794ec8ac7a4d94573b0b66a6a1c1d69bb561e6980707c8beb2f94140f)",
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

result: `Err(["Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: \"check if resource($0), operation(\\\"read\\\"), right($0, \\\"read\\\")\" })"])`


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

```

### validation

verifier code:
```
resource("file1");
operation("read");
time(2020-12-21T09:23:12+00:00);
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "revocation_id(0, hex:09fe2276d4a6f7a0cb53e4d5f804f96ecfb500d5e17004313fb3f2ce329250f2f6dca25a6af669775f8011fde7d6c00d7e6217faa5746417c328887e89837503)",
    "revocation_id(1, hex:c3a558b2a401af6de4a39a60e427fdd6692320370a3ebf54c9aef67cd6b1cd5406d60b61ef297a2a73b9a07adf62f2e0c29a43c90a126eb157057361e781bd05)",
    "time(2020-12-21T09:23:12+00:00)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 1, check_id: 1, rule: \"check if time($date), $date <= 2018-12-20T00:00:00+00:00\" })"])`


------------------------------

## verifier scope: test10_verifier_scope.bc
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

```

### validation

verifier code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:7fa94693fffd5f804deac39567c7b79ba839d961368d668cc0ea7b84a895df64a0cb8f89774fdf356066980f202ba7fd9a645e6dbe0efc3e9fadfdad4ce99907)",
    "revocation_id(1, hex:666823b6e4e465241cabca743f0d49e461bd6cb3ad04e4646f33ca187554a9fd8ad37998411abf9cfc7bf33f84cce7f34126d87c0638503520d353b7afb41505)",
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

result: `Err(["Verifier(FailedVerifierCheck { check_id: 0, rule: \"check if right($0, $1), resource($0), operation($1)\" })"])`


------------------------------

## verifier authority checks: test11_verifier_authority_caveats.bc
### token

authority:
symbols: ["file1", "read"]

```
right("file1", "read");
```

```

### validation

verifier code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:87298abf1b281814c29c4a52cf3252eddd454703edae0e2599c560ebd471c5d95b0c73cb80ba767ad29cb3af89cdb86df0f5a22ed297b4b3374d9d270751100c)",
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

result: `Err(["Verifier(FailedVerifierCheck { check_id: 0, rule: \"check if right($0, $1), resource($0), operation($1)\" })"])`


------------------------------

## authority checks: test12_authority_caveats.bc
### token

authority:
symbols: ["check1", "file1"]

```
check if resource("file1");
```

```

### validation for "file1"

verifier code:
```
resource("file1");
operation("read");
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "revocation_id(0, hex:bb673d5a10e849db2903e9cd9ca6134bcff4720628ef97b613a20a310d1b0980208ab53eb584f2be049bf7381c3fcae45ec88e7cce06f0af10ebd1e86cd9b902)",
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

verifier code:
```
resource("file2");
operation("read");
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "revocation_id(0, hex:bb673d5a10e849db2903e9cd9ca6134bcff4720628ef97b613a20a310d1b0980208ab53eb584f2be049bf7381c3fcae45ec88e7cce06f0af10ebd1e86cd9b902)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: \"check if resource(\\\"file1\\\")\" })"])`


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

```

### validation for "file1"

verifier code:
```
resource("file1");
time(2020-12-21T09:23:12+00:00);
```

verifier world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:5ba8b06cd4c4f7fe0993836ceee769ec915be987f643662ec7d8d4f244286cdf65a1adf6e5327688cb0d8a4f40ef368c11bf7c27d8507608920b0ccd2249ad0f)",
    "revocation_id(1, hex:f1128098488f48f2185539a8f1b2493e3e66cd824b0226a5d9424eea685290938aafb2b18147e9f08d64e557f2bea5954d30bf66032bd0f12b2a9d6e310ba208)",
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

verifier code:
```
resource("file2");
time(2020-12-21T09:23:12+00:00);
```

verifier world:
```
World {
  facts: {
    "resource(\"file2\")",
    "revocation_id(0, hex:5ba8b06cd4c4f7fe0993836ceee769ec915be987f643662ec7d8d4f244286cdf65a1adf6e5327688cb0d8a4f40ef368c11bf7c27d8507608920b0ccd2249ad0f)",
    "revocation_id(1, hex:f1128098488f48f2185539a8f1b2493e3e66cd824b0226a5d9424eea685290938aafb2b18147e9f08d64e557f2bea5954d30bf66032bd0f12b2a9d6e310ba208)",
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

result: `Err(["Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: \"check if valid_date($0), resource($0)\" })"])`


------------------------------

## regex_constraint: test14_regex_constraint.bc
### token

authority:
symbols: ["resource_match", "0", "file[0-9]+.txt"]

```
check if resource($0), $0.matches("file[0-9]+.txt");
```

```

### validation for "file1"

verifier code:
```
resource("file1");
```

verifier world:
```
World {
  facts: {
    "resource(\"file1\")",
    "revocation_id(0, hex:7d7317a3d4c1705ef0f14daab4b0877dee913db0883b0efb1e8af4b3e0762262a51dc6e8f179af573723fd77c919cfccc02d376d8a80abd2a33716aa99558a05)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: \"check if resource($0), $0.matches(\\\"file[0-9]+.txt\\\")\" })"])`
### validation for "file123"

verifier code:
```
resource("file123.txt");
```

verifier world:
```
World {
  facts: {
    "resource(\"file123.txt\")",
    "revocation_id(0, hex:7d7317a3d4c1705ef0f14daab4b0877dee913db0883b0efb1e8af4b3e0762262a51dc6e8f179af573723fd77c919cfccc02d376d8a80abd2a33716aa99558a05)",
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

```

### validation

verifier code:
```

check if must_be_present($0) or must_be_present($0);
```

verifier world:
```
World {
  facts: {
    "must_be_present(\"hello\")",
    "revocation_id(0, hex:a83fd5ebefd85373c624bfa0847c2c13726b1120319b735781a34fd59a6f045dc906b1ba7006e9c26687c8d5e0ba23eebd68f4a868367ee7ceb1ea377cc67409)",
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

```

### validation

verifier code:
```
```

verifier world:
```
World {
  facts: {
    "check1(\"test\")",
    "revocation_id(0, hex:75a758d48783b23b4337b71c3567fb1d5293d5538d74cf3a4f1bfe306a0f79f393f2e7e9bd48ca48ccb587deca870b71df82f7decf8ed663e801eb4ee7080804)",
    "revocation_id(1, hex:177092ffbb60e4e44ea5c7d07415782c018a28a2765317ae3e14526ca8fbb0f55a60b264c60269ac277a48a868f27774d10cd46cbe77380dad9e73c82c49eb00)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: \"check if resource(\\\"hello\\\")\" })"])`


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

```

### validation

verifier code:
```
```

verifier world:
```
World {
  facts: {
    "revocation_id(0, hex:ed59c23946d8f86642de25d718ae29ad25d923bf303bf8bd1460eee140e28e12571eadf4bd03c952af43573b1dd32e764d70dc9f76f57920c42507612b348602)",
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

```

### validation

verifier code:
```
operation("write");
```

verifier world:
```
World {
  facts: {
    "operation(\"write\")",
    "revocation_id(0, hex:814d95cb15c293aaefe111506e40ee48a6630a4409e2032288865fb3322615e6c4d2f7b64762d5a755310936ebc9314927816b5640a9c9b7cc2374bdcf649b0a)",
    "revocation_id(1, hex:08a93c775baef6d662229a7059faa307517589359d229fa90e1cc7a540361c607415257853d834fe7557d9c54005550627ee8c5d05ce031b923069f9bef71a0e)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["FailedLogic(InvalidBlockRule(0, \"operation($unbound, \\\"read\\\") <- operation($any1, $any2)\"))"])`


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

```

### validation

verifier code:
```
operation("write");
```

verifier world:
```
World {
  facts: {
    "operation(\"read\")",
    "operation(\"write\")",
    "revocation_id(0, hex:9c3f40ab693f438286e61310572c6fe0fbf5bb289cad11e5fb6425c10fdd55922a3398c3fef64e7f8da2bb86e12f76b520d70497144a1a54dc6bb2037d774e09)",
    "revocation_id(1, hex:c4956938e31a6e29f609e833884db72dc49636344f3a40c1b80a839ebdb08d2453a422d8d2a33b8950e1750607adede01c52415f85034b7b7df1886de9fc9502)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(["Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: \"check if operation(\\\"read\\\")\" })"])`
