# protoc-gen-gotag

A protoc plugin used to add/replace struct tags on generated protobuf messages.
Get  it using ```go get github.com/euforic/protoc-gen-gotag ```It supports the following features,

## Add/Replace Tags

New tags like xml, sql, bson etc... can be added to struct messages of protobuf. Example
```proto
syntax = "proto3";

package example;

import "gotag/gotag.proto";

message Example {
    string with_new_tags = 1 [(gotag.tags) = "graphql:\"withNewTags,optional\"" ];
    string with_new_multiple = 2 [(gotag.tags) = "graphql:\"withNewTags,optional\" xml:\"multi,omitempty\"" ];
    
    string replace_default = 3 [(gotag.tags) = "json:\"replacePrevious\""] ; 

    oneof one_of {
        option (gotag.oneof_tags) = "graphql:\"withNewTags,optional\"";
        string a = 5 [(gotag.tags) = "json:\"A\""];
        int32 b_jk = 6 [(gotag.tags) = "json:\"b_Jk\""];
    }
}

message SecondMessage {
    string with_new_tags = 1 [(gotag.tags) = "graphql:\"withNewTags,optional\"" ];
    string with_new_multiple = 2 [(gotag.tags) = "graphql:\"withNewTags,optional\" xml:\"multi,omitempty\"" ];
    
    string replace_default = 3 [(gotag.tags) = "json:\"replacePrevious\""] ; 
}
``` 

Then struct tags can be added by running this command **after** the regular protobuf generation command.
```bash
    protoc -I /usr/local/include \
    	-I . \
    	--gotag_out=:. example/example.proto
```

In the above example tags like graphql and xml will be added whereas existing tags such as json are replaced with the supplied values. 

## Auto add tags on field

Automatically add custom tags to message field using provided transformer.
It will compile the tag as ```tag:"snaked_key_name"``` by default if no transformer is being provide.
To provide transformer, use: ```tagName-as-transformer``` instruction when running `gotag`

```bash
    protoc -I /usr/local/include \
    	-I . \
    	--gotag_out=auto="form+db-as-camel":. example/example.proto
```

The above command will add two addtional tags (form and db) for each field. The form tag will be lower_snake_case and db tag will be lowerCamelCase

Supported transformers:

| Keys                                                     | Action                            | Ex                  | 
| -------------------------------------------------------- | --------------------------------- | ------------------- |
| "lower_snake", "lower_snake_case", "snake", "snake_case" | Make lower  snake case key        | someKey -> some_key |
| "upper_snake", "upper_snake_case"                        | Make upper snake case key         | someKey -> Some_key |
| "lower_camel", "lower_camel_case", "camel", "camel_case" | Make lower camel case key         | someKey -> someKey  |
| "upper_camel", "upper_camel_case"                        | Make upper camel case key         | someKey -> SomeKey  |
| "dot_notation", "dot", "lower_dot_notation", "lower_dot" | Make lower cased dot notation key | someKey -> some.key |
| "upper_dot", "upper_dot_notation"                        | Make upper cased dot notation key | someKey -> Some.Key | 

## Add tags to XXX* fields

It is very useful to ignore XXX* fields in protobuf generated messages. The go protocol buffer compiler adds ```json:"-"``` tag to all XXX* fields. Additional tags can be added to these fields using the 'xxx' option of PGGT. It can be done like this. All '+' characters will be replaced with ':'.

```bash
    protoc -I /usr/local/include \
    	-I . \
    	--gotag_out=xxx="graphql+\"-\" bson+\"-\"":. example/example.proto
```

### Note
 
 This should always run after protocol buffer compiler has run. The command such as the one below will fail/produce unexpected results.
 ```bash
    protoc -I /usr/local/include \
        	-I . \
        	--go_out=:. \
        	--gotag_out=xxx="graphql+\"-\" bson+\"-\"":. example/example.proto
``` 


### Credit

Originally based on [protoc-gen-gotag](github.com/srikrsna/protoc-gen-gotag)
