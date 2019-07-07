#  插件列表

插件详情查阅 https://developers.eos.io/eosio-nodeos/docs/<插件名称>

- chain_api_plugin：chain_api_plugin exposes functionality from the chain_plugin to the RPC API interface managed by the http_plugin
  ```c++
  // hpp文件主要定义
  virtual void set_program_options(options_description&, options_description&) override;
void plugin_initialize(const variables_map&);
  void plugin_startup();
  void plugin_shutdown();
  ```
- chain_plugin 链基础功能API
  ```shell
  Config Options for eosio::chain_plugin:
    --blocks-dir arg (="blocks")          the location of the blocks directory
                                          (absolute path or relative to
                                          application data dir)
    --checkpoint arg                      Pairs of [BLOCK_NUM,BLOCK_ID] that
                                          should be enforced as checkpoints.
    --wasm-runtime wavm/wabt              Override default WASM runtime
    --abi-serializer-max-time-ms arg (=15000)
                                          Override default maximum ABI
                                          serialization time allowed in ms
    --chain-state-db-size-mb arg (=1024)  Maximum size (in MiB) of the chain
                                          state database
    --chain-state-db-guard-size-mb arg (=128)
                                          Safely shut down node when free space
                                          remaining in the chain state database
                                          drops below this size (in MiB).
    --reversible-blocks-db-size-mb arg (=340)
                                          Maximum size (in MiB) of the reversible
                                          blocks database
    --reversible-blocks-db-guard-size-mb arg (=2)
                                          Safely shut down node when free space
                                          remaining in the reverseible blocks
                                          database drops below this size (in
                                          MiB).
    --chain-threads arg (=2)              Number of worker threads in controller
                                          thread pool
    --contracts-console                   print contract's output to console
    --actor-whitelist arg                 Account added to actor whitelist (may
                                          specify multiple times)
    --actor-blacklist arg                 Account added to actor blacklist (may
                                          specify multiple times)
    --contract-whitelist arg              Contract account added to contract
                                          whitelist (may specify multiple times)
    --contract-blacklist arg              Contract account added to contract
                                          blacklist (may specify multiple times)
    --action-blacklist arg                Action (in the form code::action) added
                                          to action blacklist (may specify
                                          multiple times)
    --key-blacklist arg                   Public key added to blacklist of keys
                                          that should not be included in
                                          authorities (may specify multiple
                                          times)
    --sender-bypass-whiteblacklist arg    Deferred transactions sent by accounts
                                          in this list do not have any of the
                                          subjective whitelist/blacklist checks
                                          applied to them (may specify multiple
                                          times)
    --read-mode arg (=speculative)        Database read mode ("speculative",
                                          "head", or "read-only").
                                          In "speculative" mode database contains
                                          changes done up to the head block plus
                                          changes made by transactions not yet
                                          included to the blockchain.
                                          In "head" mode database contains
                                          changes done up to the current head
                                          block.
                                          In "read-only" mode database contains
                                          incoming block changes but no
                                          speculative transaction processing.
  
    --validation-mode arg (=full)         Chain validation mode ("full" or
                                          "light").
                                          In "full" mode all incoming blocks will
                                          be fully validated.
                                          In "light" mode all incoming blocks
                                          headers will be fully validated;
                                          transactions in those validated blocks
                                          will be trusted
  
    --disable-ram-billing-notify-checks   Disable the check which subjectively
                                          fails a transaction if a contract bills
                                          more RAM to another account within the
                                          context of a notification handler (i.e.
                                          when the receiver is not the code of
                                          the action).
    --trusted-producer arg                Indicate a producer whose blocks
                                          headers signed by it will be fully
                                          validated, but transactions in those
                                          validated blocks will be trusted.
  
  Command Line Options for eosio::chain_plugin:
    --genesis-json arg                    File to read Genesis State from
    --genesis-timestamp arg               override the initial timestamp in the
                                          Genesis State file
    --print-genesis-json                  extract genesis_state from blocks.log
                                          as JSON, print to console, and exit
    --extract-genesis-json arg            extract genesis_state from blocks.log
                                          as JSON, write into specified file, and
                                          exit
    --fix-reversible-blocks               recovers reversible block database if
                                          that database is in a bad state
    --force-all-checks                    do not skip any checks that can be
                                          skipped while replaying irreversible
                                          blocks
    --disable-replay-opts                 disable optimizations that specifically
                                          target replay
    --replay-blockchain                   clear chain state database and replay
                                          all blocks
    --hard-replay-blockchain              clear chain state database, recover as
                                          many blocks as possible from the block
                                          log, and then replay those blocks
    --delete-all-blocks                   clear chain state database and block
                                          log
    --truncate-at-block arg (=0)          stop hard replay / block log recovery
                                          at this block number (if set to
                                          non-zero number)
    --import-reversible-blocks arg        replace reversible block database with
                                          blocks imported from specified file and
                                          then exit
    --export-reversible-blocks arg        export reversible block database in
                                          portable format into specified file and
                                          then exit
    --snapshot arg
  ```
- db_size_api_plugin 数据库size插件接口
  - db_size_index_count, (index)(row_count) 
  - db_size_stats, (free_bytes)(used_bytes)(size)(indices) 
- faucet_testnet_plugin EOS测试网的令牌自动化
- history_api_plugin  
- history_plugin  搜集所有的交易信息，包括收集所有traces信息  [插件介绍](https://dpos.club/topics/297)  
- http_client_plugin：http_client_plugin 是一个内部实用程序插件,为producer_plugin提供能力，使producer_plugin能安全地使用外部 keosd 实例为块签名。只有当producer_plugin被配置生产块的时候才能使用它。
- http_plugin node启动和管理RPC API的核心插件
  ```shell
   --http-server-address arg (=127.0.0.1:8888)
                                          The local IP and port to listen for
                                          incoming http connections; set blank to
                                          disable.
    --https-server-address arg            The local IP and port to listen for
                                          incoming https connections; leave blank
                                          to disable.
    --https-certificate-chain-file arg    Filename with the certificate chain to
                                          present on https connections. PEM
                                          format. Required for https.
    --https-private-key-file arg          Filename with https private key in PEM
                                          format. Required for https
    --access-control-allow-origin arg     Specify the Access-Control-Allow-Origin
                                          to be returned on each request.
    --access-control-allow-headers arg    Specify the Access-Control-Allow-Header
                                          s to be returned on each request.
    --access-control-max-age arg          Specify the Access-Control-Max-Age to
                                          be returned on each request.
    --access-control-allow-credentials    Specify if Access-Control-Allow-Credent
                                          ials: true should be returned on each
                                          request.
    --max-body-size arg (=1048576)        The maximum body size in bytes allowed
                                          for incoming RPC requests
    --verbose-http-errors                 Append the error log to HTTP responses
    --http-validate-host arg (=1)         If set to false, then any incoming
                                          "Host" header is considered valid
    --http-alias arg                      Additionaly acceptable values for the
                                          "Host" header of incoming HTTP
                                          requests, can be specified multiple
                                          times.  Includes http/s_server_address
                                          by default.
  ```
- login_plugin 验证用户是否有权限对指定授权进行签名
- mongo_db_plugin 使用MongoDB管理EOS输出的json数据
- net_api_plugin
- net_plugin p2p网络协议
- producer_api_plugin
- producer_api_plugin 
- producer_plugin 挖矿
- state_history_plugin 将链上的数据缓存为file，插件监听app连接的socket，将数据发送到app上。典型应用**fill-postgresql**（现已合并到history_tool中）
- template_plugin 空
- test_control_api_plugin 
- test_control_plugin 测试一个nodeos实例（节点）是否关闭
- txn_test_gen_plugin 此插件提供了一种针对货币协定每秒生成给定交易量的方法
- wallet_api_plugin
- wallet_plugin 

> **wallet_plugin** is not designed to be loaded as a plugin on a publicly accessible node without further security measures. This is particularly true when also loading the **wallet_api_plugin**, which should not under any conditions be loaded on a publicly accessible node.