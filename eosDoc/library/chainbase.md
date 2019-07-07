# Chainbase 数据库

交易型数据库 transactional database

##特点

- 支持多对象，多目录
- 状态持久化，多线程共享
- 嵌套事务写入,能够撤消更改

## 核心文件

- chainbase/chainbase.hpp
- src/chainbase.cpp

## 使用 example

```c++
num tables {
   book_table
};

/**
 * Defines a "table" for storing books. This table is assigned a
 * globally unique ID (book_table) and must inherit from chainbase::object<> which
 * decorates the book type by defining "id_type" and "type_id"
 */
/**
定义一个table， 且用一个全局唯一的ID(book_table)标记
tabele 必须继承chainbase::object<>
*/
struct book : public chainbase::object<book_table, book> {

   /** defines a default constructor for types that don't have
     * members requiring dynamic memory allocation.
     */
  /**
  定义默认book构造器
  */
   CHAINBASE_DEFAULT_CONSTRUCTOR( book )

   id_type          id; ///< this manditory member is a primary key
   int pages        = 0;
   int publish_date = 0;
};

struct by_id;
struct by_pages;
struct by_date;

/**
 * This is a relatively standard boost multi_index_container definition that has three
 * requirements to be used withn a chainbase database:
 *   - it must use chainbase::allocator<T>
 *   - the first index must be on the primary key (id) and must be unique (hashed or ordered)
 */
typedef multi_index_container<
  book,
  indexed_by<
     ordered_unique< tag<by_id>, member<book,book::id_type,&book::id> >, ///< required
     ordered_non_unique< tag<by_pages>, BOOST_MULTI_INDEX_MEMBER(book,int,pages) >,
     ordered_non_unique< tag<by_date>, BOOST_MULTI_INDEX_MEMBER(book,int,publish_date) >
  >,
  chainbase::allocator<book> ///< required for use with chainbase::database
> book_index;

/**
    This simple program will open database_dir and add two new books every time
    it is run and then print out all of the books in the database.
 */
int main( int argc, char** argv ) {
   chainbase::database db;
   db.open( "database_dir", database::read_write, 1024*1024*8 ); /// open or create a database with 8MB capacity
   db.add_index< book_index >(); /// open or create the book_index


   const auto& book_idx = db.get_index<book_index>().indicies();

   /**
      Returns a const reference to the book, this pointer will remain
      valid until the book is removed from the database.
    */
   const auto& new_book300 = db.create<book>( [&]( book& b ) {
       b.pages = 300+book_idx.size();
   } );
   const auto& new_book400 = db.create<book>( [&]( book& b ) {
       b.pages = 300+book_idx.size();
   } );

   /**
      You modify a book by passing in a lambda that receives a
      non-const reference to the book you wish to modify.
   */
   db.modify( new_book300, [&]( book& b ) {
      b.pages++;
   });

   for( const auto& b : book_idx ) {
      std::cout << b.pages << "\n";
   }

   auto itr = book_idx.get<by_pages>().lower_bound( 100 );
   if( itr != book_idx.get<by_pages>().end() ) {
      std::cout << itr->pages;
   }

   db.remove( new_book400 );

   return 0;
}
```

## 方法罗列

eos database的实现源码在 *eos/libraries/chainbase*目录下的chainbase.hpp，chainbase.cpp。database的主要是实现是通过*boost::multi_index_container* 和 *boost::interprocess::managed_mapped_file*，multi_index_container 用来做容器实现增删改查，managed_mapped_file实现内存映射文件并为multi_index_container 提供内存分配器allocator。

- multi_index_container：boost实现的一个多个关键字索引容器，通俗的理解你把它看成一张数据库中的表，eos中存在的表一共有（搜索add_index)30多张不同类型的multi_index_container表，其中eos合约开发数据库操作相关的有2张表 *table_id_multi_index*和*key_value_index*， eos合约下面的eosiolib/multi_index.hpp仅仅是给eos合约开发的一个数据库操作接口，最终的所有操作都会映射到table_id_multi_index，key_value_index这2个multi_index_container中去， 这2张表，table_id_multi_index保存着所有eos合约中创建的表信息，key_value_index保存着所有eos合约中创建的表记录（在eos合约中不同表创建的记录）最终都会保存到该表中。
- undo session：undo session这个概念是eos建立在multi_index_container之上用于执行回滚操作，实现的大体思路为：**当创建一个undo session之后对增，删，改操作进行跟踪，用户如果增加了记录row，就记录该记录row的id，好在undo的时候删除该记录row，用户如果删除了记录row，就记录该记录row，好在undo的时候还原该row，用户如果修改了记录row，就保存修改之前的记录row，好在undo的时候还原该row。当用户执行undo的时候用保存的undo状态信息还原，当用户进行commit的时候抛弃保存的undo状态信息使修改变得不可逆**
- database中的增删改查和undo session主要用过generic_index类实现

### chainbase.hpp文件

```c++
// 数据库操作类以及常用属性
class generic_index{
  void validate()const{...} //校验可执行的内存空间
  template<typename Constructor>  // 增
         const value_type& emplace( Constructor&& c ) {}
  template<typename Modifier> // 改
         void modify( const value_type& obj, Modifier&& m ) {}
  void remove( const value_type& obj ) {...} //删
  template<typename CompatibleKey> //  返回指针，主要支持get方法
         const value_type* find( CompatibleKey&& key )const {...}
  template<typename CompatibleKey> // 查
         const value_type& get( CompatibleKey&& key )const {...}
}

class session{
  /** leaves the UNDO state on the stack when session goes out of scope */
  void push()   { _apply = false; }
  /**
  *  This method works similar to git squash, it merges the change set from the two most
  *  recent revision numbers into one revision number (reducing the head revision number)
  *
  *  This method does not change the state of the index, only the state of the undo buffer.
  */
  void squash() { if( _apply ) _index.squash(); _apply = false; }
  /**
  *  Restores the state to how it was prior to the current session discarding all changes
  *  made between the last revision and the current revision.
  */
  void undo()   { if( _apply ) _index.undo();  _apply = false; }
  session start_undo_session( bool enabled ) {...}
  /**
  * Discards all undo history prior to revision
  */
  void commit( int64_t revision ){...}
  /**
  * Unwinds all undo states
  */
  void undo_all(){...}
  /** "cannot set revision while there is an existing undo stack" */
  void set_revision( uint64_t revision ){...}
  
  void remove_object( int64_t id ){...}
  std::pair<int64_t, int64_t> undo_stack_revision_range()const {...}
  
  /**
  *  Each new session increments the revision, a squash will decrement the revision by combining
  *  the two most recent revisions into one revision.
  *
  *  Commit will discard all revisions prior to the committed revision.
  */
}

class read_write_mutex_manager{  //读写互斥管理，lock
  void next_lock(){}
	read_write_mutex& current_lock(){}
  uint32_t current_lock_num(){}
}

class database{
  void set_revision( uint64_t revision ){}
template<typename MultiIndexType>
         void add_index() {...}
	void undo();
	void squash();
	void commit( int64_t revision );
	void undo_all();
  void set_revision( uint64_t revision ){}
  template<typename MultiIndexType>
         void add_index() {}
  /**
  * 省略。。。
  */
  
  
}

```

### chainbase.cpp

```c++
namespace chainbase {
  // ...
   database::~database()
   {...}

   void database::set_require_locking( bool enable_require_locking )
   {
#ifdef CHAINBASE_CHECK_LOCKING
      _enable_require_locking = enable_require_locking;
#endif
   }

#ifdef CHAINBASE_CHECK_LOCKING
   void database::require_lock_fail( const char* method, const char* lock_type, const char* tname )const
   {
      std::string err_msg = "database::" + std::string( method ) + " require_" + std::string( lock_type ) + "_lock() failed on type " + std::string( tname );
      std::cerr << err_msg << std::endl;
      BOOST_THROW_EXCEPTION( std::runtime_error( err_msg ) );
   }
#endif

   void database::undo()
   {
      for( auto& item : _index_list )
      {
         item->undo();
      }
   }

   void database::squash()
   {
      for( auto& item : _index_list )
      {
         item->squash();
      }
   }

   void database::commit( int64_t revision )
   {
      for( auto& item : _index_list )
      {
         item->commit( revision );
      }
   }

   void database::undo_all()
   {
      for( auto& item : _index_list )
      {
         item->undo_all();
      }
   }

   database::session database::start_undo_session( bool enabled )
   {
      if( enabled ) {
         vector< std::unique_ptr<abstract_session> > _sub_sessions;
         _sub_sessions.reserve( _index_list.size() );
         for( auto& item : _index_list ) {
            _sub_sessions.push_back( item->start_undo_session( enabled ) );
         }
         return session( std::move( _sub_sessions ) );
      } else {
         return session();
      }
   }
}  // namespace chainbase
```



