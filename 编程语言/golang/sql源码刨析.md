

~~~golang
# database.go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
	"go.uber.org/zap"
)

var logx *zap.Logger

func NewConnection(driver, dataSource string) (*sql.DB, error) {
	return sql.Open(driver, dataSource)
}
~~~

> sql.open打开一个db连接池

~~~golang
# database/sql 
var (
	driversMu sync.RWMutex
	drivers   = make(map[string]driver.Driver)
)

func Open(driverName, dataSourceName string) (*DB, error) {
	driversMu.RLock()
	driveri, ok := drivers[driverName]
	driversMu.RUnlock()
	if !ok {
		return nil, fmt.Errorf("sql: unknown driver %q (forgotten import?)", driverName)
	}

	if driverCtx, ok := driveri.(driver.DriverContext); ok {
		connector, err := driverCtx.OpenConnector(dataSourceName)
		if err != nil {
			return nil, err
		}
		return OpenDB(connector), nil
	}

	return OpenDB(dsnConnector{dsn: dataSourceName, driver: driveri}), nil
}
~~~

> 查看open方法，源码第二行，drivers[driverName], 可是drivers在sql中并没有初始化，难道这里得到nil？显然不是的。在上面的database.go中，import了_ "github.com/go-sql-driver/mysql",这个包进行了初始化


~~~golang
# github.com/go-sql-driver/mysql      driver.go
func init() {
	sql.Register("mysql", &MySQLDriver{})
}


# database/sql
func Register(name string, driver driver.Driver) {
	driversMu.Lock()
	defer driversMu.Unlock()
	if driver == nil {
		panic("sql: Register driver is nil")
	}
	if _, dup := drivers[name]; dup {
		panic("sql: Register called twice for driver " + name)
	}
	drivers[name] = driver
}
~~~

> 继续看database/sql的open方法，在获取到driver之后,调用了DriverContext.openConnector
~~~golang

func open ... {

	.....
	if driverCtx, ok := driveri.(driver.DriverContext); ok {
		connector, err := driverCtx.OpenConnector(dataSourceName)
		if err != nil {
			return nil, err
		}
		return OpenDB(connector), nil
	}
}


// If a Driver implements DriverContext, then sql.DB will call
// OpenConnector to obtain a Connector and then invoke
// that Connector's Connect method to obtain each needed connection,
// instead of invoking the Driver's Open method for each connection.
// The two-step sequence allows drivers to parse the name just once
// and also provides access to per-Conn contexts.
// 如果一个driver实现了DriverContext接口，那么sql.DB会调用OpenConnector方法取
// 一个连接器，然后调用连接器的connect的方法取获取连接，而不是为每一个连接都调用Open方法
// 这两步允许driver只解析一次name，也提供对连接上下文的访问
type DriverContext interface {
	// OpenConnector must parse the name in the same format that Driver.Open
	// parses the name parameter.
	OpenConnector(name string) (Connector, error)
}
~~~~
+ 其中DriverContext是一个接口
+ github/go-sql-driver/mysql实现了该接口方法OpenConnector

~~~golang
# github/go-sql-driver/mysql    driver.go

// OpenConnector implements driver.DriverContext.
func (d MySQLDriver) OpenConnector(dsn string) (driver.Connector, error) {
	cfg, err := ParseDSN(dsn)
	if err != nil {
		return nil, err
	}
	// 这里返回的本包中的connector，但是参数定义的是database/sql中的driver.Connector
	// 所以本包的connector需要实现database/sql中的driver.Connector中的方法来实现多态// 调用
	return &connector{
		cfg: cfg,
	}, nil
}

type connector struct {
	cfg *Config // immutable private copy.
}
~~~


















~~~golang
// DB is a database handle representing a pool of zero or more
// underlying connections. It's safe for concurrent use by multiple
// goroutines.
// The sql package creates and frees connections automatically; it
// also maintains a free pool of idle connections. If the database has
// a concept of per-connection state, such state can be reliably observed
// within a transaction (Tx) or connection (Conn). 
//
// Once DB.Begin is called, the
// returned Tx is bound to a single connection. Once Commit or
// Rollback is called on the transaction, that transaction's
// connection is returned to DB's idle connection pool. The pool size
// can be controlled with SetMaxIdleConns.
type DB struct {
	// Atomic access only. At top of struct to prevent mis-alignment
	// on 32-bit platforms. Of type time.Duration.
	waitDuration int64 // Total time waited for new connections.

	connector driver.Connector
	// numClosed is an atomic counter which represents a total number of
	// closed connections. Stmt.openStmt checks it before cleaning closed
	// connections in Stmt.css.
	numClosed uint64

	mu           sync.Mutex // protects following fields
	freeConn     []*driverConn
	connRequests map[uint64]chan connRequest
	nextRequest  uint64 // Next key to use in connRequests.
	numOpen      int    // number of opened and pending open connections
	// Used to signal the need for new connections
	// a goroutine running connectionOpener() reads on this chan and
	// maybeOpenNewConnections sends on the chan (one send per needed connection)
	// It is closed during db.Close(). The close tells the connectionOpener
	// goroutine to exit.
	openerCh          chan struct{}
	closed            bool
	dep               map[finalCloser]depSet
	lastPut           map[*driverConn]string // stacktrace of last conn's put; debug only
	maxIdleCount      int                    // zero means defaultMaxIdleConns; negative means 0
	maxOpen           int                    // <= 0 means unlimited
	maxLifetime       time.Duration          // maximum amount of time a connection may be reused
	maxIdleTime       time.Duration          // maximum amount of time a connection may be idle before being closed
	cleanerCh         chan struct{}
	waitCount         int64 // Total number of connections waited for.
	maxIdleClosed     int64 // Total number of connections closed due to idle count.
	maxIdleTimeClosed int64 // Total number of connections closed due to idle time.
	maxLifetimeClosed int64 // Total number of connections closed due to max connection lifetime limit.

	stop func() // stop cancels the connection opener and the session resetter.
}

~~~

+ DB是一个数据库持有0-多个连接池句柄。在多协程下是并发安全的
+ 一旦执行了db.begin, 返回的tx事务对象会被绑定到单连接下。
+ 一旦执行了tx.commit/tx.rollback，那么事务所绑定的连接会被释放到空闲的数据库连接池中
+ 数据库的连接池通过方法SetMaxIdleConns进行控制
