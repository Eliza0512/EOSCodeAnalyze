# 库结构

- include/eosio/chain
- webassembly
- CMakeLists.txt
- abi_serializer.cpp
- apply_context.cpp
- asset.cpp
- authorization_manager.cpp
- block_header.cpp
- block_header_state.cpp
- block_log.cpp
- block_state.cpp
- chain_config.cpp
- chain_id_type.cpp
- controller.cpp
- eosio_contract.cpp
- eosio_contract_abi.cpp
- fork_database.cpp
- genesis_intrinsics.cpp
- genesis_state.cpp
- genesis_state_root_key.cpp.in
- merkle.cpp
- name.cpp
- protocol_feature_activation.cpp
- protocol_feature_manager.cpp
- protocol_state_object.cpp
- resource_limits.cpp
- snapshot.cpp
- thread_utils.cpp
- trace.cpp
- transaction.cpp
- transaction_context.cpp
- transaction_metadata.cpp
- wasm_eosio_binary_ops.cpp
- wasm_eosio_injection.cpp
- wasm_eosio_validation.cpp
- wasm_interface.cpp
- wast_to_wasm.cpp
- whitelisted_intrinsics.cpp

**注：虚拟机相关wasm、wavm相关等不做分析**

# include/eosio/chain

## 基础对象 

- types.hpp  eos基本对象

- name.hpp 账户名称

- merkle.hpp 默克尔树

- exceptions.hpp 错误处理

- controller.hpp 控制器

- abi_def.hpp  二进制执行接口定义
  - abi结构
    ```c++
    struct abi_def {
       abi_def() = default;
       abi_def(const vector<type_def>& types, const vector<struct_def>& structs, const vector<action_def>& actions, const vector<table_def>& tables, const vector<clause_pair>& clauses, const vector<error_message>& error_msgs)
       :types(types)
       ,structs(structs)
       ,actions(actions)
       ,tables(tables)
       ,ricardian_clauses(clauses)
       ,error_messages(error_msgs)
       {}
    
       string                              version = "";
       vector<type_def>                    types;
       vector<struct_def>                  structs;
       vector<action_def>                  actions;
       vector<table_def>                   tables;
       vector<clause_pair>                 ricardian_clauses;
       vector<error_message>               error_messages;
       extensions_type                     abi_extensions;
       may_not_exist<vector<variant_def>>  variants;
    };
    ```
  - 方法
    ```c++
    template<typename ST, typename T>
    datastream<ST>& operator << (datastream<ST>& s, const eosio::chain::may_not_exist<T>& v) {}
    
    template<typename T>
    void to_variant(const eosio::chain::may_not_exist<T>& e, fc::variant& v) {
       to_variant( e.value, v);
    }
    
    template<typename T>
    void from_variant(const fc::variant& v, eosio::chain::may_not_exist<T>& e) {
       from_variant( v, e.value );
    }
    ```
  
- abi_serialization.hpp abi接口序列化相关的操作，这里不做详细展开


## 创世区块 genesis

- genesis_intrinsics.hpp
- genesis_state.hpp

## 交易 transaction
- action.hpp
  ```c++
     struct action {
        account_name               account;
        action_name                name;
        vector<permission_level>   authorization;
        bytes                      data;
  
        action(){}
  
        template<typename T, std::enable_if_t<std::is_base_of<bytes, T>::value, int> = 1>
        action( vector<permission_level> auth, const T& value ) {
           account     = T::get_account();
           name        = T::get_name();
           authorization = move(auth);
           data.assign(value.data(), value.data() + value.size());
        }
  
        template<typename T, std::enable_if_t<!std::is_base_of<bytes, T>::value, int> = 1>
        action( vector<permission_level> auth, const T& value ) {
           account     = T::get_account();
           name        = T::get_name();
           authorization = move(auth);
           data        = fc::raw::pack(value);
        }
  
        action( vector<permission_level> auth, account_name account, action_name name, const bytes& data )
              : account(account), name(name), authorization(move(auth)), data(data) {
        }
  
        template<typename T>
        T data_as()const {
           EOS_ASSERT( account == T::get_account(), action_type_exception, "account is not consistent with action struct" );
           EOS_ASSERT( name == T::get_name(), action_type_exception, "action name is not consistent with action struct"  );
           return fc::raw::unpack<T>(data);
        }
     };
  ```
- action_receipt.hpp 每个操作执行后都会收到一个消息回执
- transaction.hpp
- transaction_context.hpp
- transaction_metadata.hpp
- transaction_object.hpp

## 链 chain

- chain_config.hpp
- chain_id_type.hpp
- chain_snapshot.hpp

## 区块 block

- block.hpp
- block_timestamp.hpp
- block_state.hpp
- block_header.hpp
- block_header_state.hpp
- block_log.hpp
- block_summary_object.hpp

## 账户和权限管理 account & authority

[背景知识](https://developers.eos.io/eosio-nodeos/docs/accounts-and-permissions)

- account_object.hpp
- authority_checker.hpp 对每一个操作的权限检查
- authority.hpp 权限
  ``` c++
  FC_REFLECT(eosio::chain::permission_level_weight, (permission)(weight) )
  FC_REFLECT(eosio::chain::key_weight, (key)(weight) )
  FC_REFLECT(eosio::chain::wait_weight, (wait_sec)(weight) )
  FC_REFLECT(eosio::chain::authority, (threshold)(keys)(accounts)(waits) )
  FC_REFLECT(eosio::chain::shared_authority, (threshold)(keys)(accounts)(waits) )
  //确保每一个许可中的每一个key值都是唯一且排好序的，并且具体的某一项操作权限要求是可以被满足的 
  template<typename Authority>
  inline bool validate( const Authority& auth ) {...}
  ```
- authority_manager.hpp 授权管理
- permission_link_object.hpp
- permission_object.hpp 

## 资产 asset

- asset.hpp
> asset includes amount and currency symbol
>
> asset::from_string takes a string of the form "10.0000 CUR" and constructs an asset 
>
> with amount = 10 and symbol(4,"CUR")

- 