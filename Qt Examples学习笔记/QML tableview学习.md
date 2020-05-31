### 路径\quickcontrols\controls\tableview

```C++
/**代码 main.cpp**/
#include "qtquickcontrolsapplication.h"
#include "sortfilterproxymodel.h"
#include <QtQml/qqmlapplicationengine.h>
#include <QtGui/qsurfaceformat.h>
#include <QtQml/qqml.h>

int main(int argc, char *argv[])
{
    QtQuickControlsApplication app(argc, argv); //使用的条件编译 参见 代码2
    //判断程序启动的参数是否包含 "--coreprofile"
    if (QCoreApplication::arguments().contains(QLatin1String("--coreprofile")))
    {
        QSurfaceFormat fmt; // QSurface
        //设置需要的OpenGL的版本
        fmt.setVersion(4, 4);
        //QSurfaceFormat::CoreProfile ------ OpenGL 3.0版本中不支持的功能不可用。
        //设置所需的OpenGL上下文配置文件。
        //如果请求的OpenGL版本小于3.2，则忽略此设置。
        fmt.setProfile(QSurfaceFormat::CoreProfile);
        QSurfaceFormat::setDefaultFormat(fmt);
    }
    //这个模板函数在QML系统中注册c++类型。
    //QML系统中不能直接创建C++类的实例。必须通过注册的方式，给QML调用
    qmlRegisterType<SortFilterProxyModel>("org.qtproject.example", 1, 0, "SortFilterProxyModel");
    
    QQmlApplicationEngine engine(QUrl("qrc:/main.qml"));
    return app.exec();
}

```

```c++
/**代码2**/
//条件编译中  定义不同的宏 
#ifdef QT_WIDGETS_LIB
#define QtQuickControlsApplication QApplication
#else
#define QtQuickControlsApplication QGuiApplication
#endif  
```

继承关系：QApplication <- QGuiApplication <- QCoreApplication <- QObject
应用场景：如果你的应用程序是无界面的，直接使用QCoreApplication即可，如果是gui相关，但没有使用widgets模块的就使用QGuiApplication，否则使用QApplication。

```qml
TableViewColumn {
            id: titleColumn
            title: "Title"
            role: "title" //列的模型角色。
            movable: false
            resizable: false //确定列不变化大小。
            width: tableView.viewport.width - authorColumn.width - ageColumn.width
        }
```

```qml
model: SortFilterProxyModel {
            id: proxyModel
            source: sourceModel.count > 0 ? sourceModel : null
            //返回排序指示符的顺序。如果没有排序指示符，则此函数的返回值是未定义的。
            sortOrder: tableView.sortIndicatorOrder
            
            sortCaseSensitivity: Qt.CaseInsensitive //忽略大小写
            sortRole: sourceModel.count > 0 ? tableView.getColumn(tableView.sortIndicatorColumn).role : ""

            filterString: "*" + searchBox.text + "*" //筛选的内容设置
            filterSyntax: SortFilterProxyModel.Wildcard
            filterCaseSensitivity: Qt.CaseInsensitive
        }

```

```c++
//sortfilterproxymodel.h 文件
#ifndef SORTFILTERPROXYMODEL_H
#define SORTFILTERPROXYMODEL_H

#include <QtCore/qsortfilterproxymodel.h>
#include <QtQml/qqmlparserstatus.h>
#include <QtQml/qjsvalue.h>

class SortFilterProxyModel : public QSortFilterProxyModel, public QQmlParserStatus
{
    Q_OBJECT
    //Q_INTERFACES这个宏告诉Qt这个类实现了哪些接口。这是在实现插件时使用的。
    //QQmlParserStatus类提供QML解析器状态的更新
    //QQmlParserStatus为QQmlEngine实例化的类提供了一种机制，用于在类创建的关键点接收通知
    Q_INTERFACES(QQmlParserStatus)
    //修改count 属性的时候  发出 countChanged的信号
    Q_PROPERTY(int count READ count NOTIFY countChanged)
    Q_PROPERTY(QObject *source READ source WRITE setSource)

    Q_PROPERTY(QByteArray sortRole READ sortRole WRITE setSortRole)
    Q_PROPERTY(Qt::SortOrder sortOrder READ sortOrder WRITE setSortOrder)

    Q_PROPERTY(QByteArray filterRole READ filterRole WRITE setFilterRole)
    Q_PROPERTY(QString filterString READ filterString WRITE setFilterString)
    Q_PROPERTY(FilterSyntax filterSyntax READ filterSyntax WRITE setFilterSyntax)

    Q_ENUMS(FilterSyntax)//此宏向元对象系统注册一个或多个枚举类型。

public:
    explicit SortFilterProxyModel(QObject *parent = 0);

    QObject *source() const;
    void setSource(QObject *source);

    QByteArray sortRole() const;
    void setSortRole(const QByteArray &role);

    void setSortOrder(Qt::SortOrder order);

    QByteArray filterRole() const;
    void setFilterRole(const QByteArray &role);

    QString filterString() const;
    void setFilterString(const QString &filter);

    enum FilterSyntax {
        RegExp,
        Wildcard,
        FixedString
    };

    FilterSyntax filterSyntax() const;
    void setFilterSyntax(FilterSyntax syntax);

    int count() const;
    Q_INVOKABLE QJSValue get(int index) const;

    void classBegin();
    void componentComplete();

signals:
    void countChanged();

protected:
    int roleKey(const QByteArray &role) const;
    QHash<int, QByteArray> roleNames() const;
    bool filterAcceptsRow(int sourceRow, const QModelIndex &sourceParent) const;

private:
    bool m_complete;
    QByteArray m_sortRole;
    QByteArray m_filterRole;
};

#endif // SORTFILTERPROXYMODEL_H

```

```c++
//sortfilterproxymodel.cpp 文件
#include "sortfilterproxymodel.h"
#include <QtDebug>
#include <QtQml>

SortFilterProxyModel::SortFilterProxyModel(QObject *parent) : QSortFilterProxyModel(parent), m_complete(false)
{
    //信号传递 当发出rowsInserted，rowsRemoved的信号后，传递给countChanged信号
    //信号连接
    connect(this, &QSortFilterProxyModel::rowsInserted, this, &SortFilterProxyModel::countChanged);
    connect(this, &QSortFilterProxyModel::rowsRemoved, this, &SortFilterProxyModel::countChanged);
}

int SortFilterProxyModel::count() const
{
    return rowCount();
}

QObject *SortFilterProxyModel::source() const
{
    return sourceModel();
}

void SortFilterProxyModel::setSource(QObject *source)
{
    setSourceModel(qobject_cast<QAbstractItemModel *>(source));
}

QByteArray SortFilterProxyModel::sortRole() const
{
    return m_sortRole;
}

void SortFilterProxyModel::setSortRole(const QByteArray &role)
{
    if (m_sortRole != role) {
        m_sortRole = role;
        if (m_complete)
            QSortFilterProxyModel::setSortRole(roleKey(role));
    }
}

void SortFilterProxyModel::setSortOrder(Qt::SortOrder order)
{
    QSortFilterProxyModel::sort(0, order);
}

QByteArray SortFilterProxyModel::filterRole() const
{
    return m_filterRole;
}

void SortFilterProxyModel::setFilterRole(const QByteArray &role)
{
    if (m_filterRole != role) {
        m_filterRole = role;
        if (m_complete)
            QSortFilterProxyModel::setFilterRole(roleKey(role));
    }
}

QString SortFilterProxyModel::filterString() const
{
    return filterRegExp().pattern();
}
//将设置筛选的项目
void SortFilterProxyModel::setFilterString(const QString &filter)
{
    setFilterRegExp(QRegExp(filter, filterCaseSensitivity(), static_cast<QRegExp::PatternSyntax>(filterSyntax())));
}

SortFilterProxyModel::FilterSyntax SortFilterProxyModel::filterSyntax() const
{
    return static_cast<FilterSyntax>(filterRegExp().patternSyntax());
}

void SortFilterProxyModel::setFilterSyntax(SortFilterProxyModel::FilterSyntax syntax)
{
    setFilterRegExp(QRegExp(filterString(), filterCaseSensitivity(), static_cast<QRegExp::PatternSyntax>(syntax)));
}

QJSValue SortFilterProxyModel::get(int idx) const
{
    QJSEngine *engine = qmlEngine(this);
    QJSValue value = engine->newObject();
    if (idx >= 0 && idx < count()) {
        QHash<int, QByteArray> roles = roleNames();
        QHashIterator<int, QByteArray> it(roles);
        while (it.hasNext()) {
            it.next();
            value.setProperty(QString::fromUtf8(it.value()), data(index(idx, 0), it.key()).toString());
        }
    }
    return value;
}

void SortFilterProxyModel::classBegin()
{
}

void SortFilterProxyModel::componentComplete()
{
    m_complete = true;
    if (!m_sortRole.isEmpty())
        QSortFilterProxyModel::setSortRole(roleKey(m_sortRole));
    if (!m_filterRole.isEmpty())
        QSortFilterProxyModel::setFilterRole(roleKey(m_filterRole));
}

int SortFilterProxyModel::roleKey(const QByteArray &role) const
{
    QHash<int, QByteArray> roles = roleNames();
    QHashIterator<int, QByteArray> it(roles);
    while (it.hasNext()) {
        it.next();
        if (it.value() == role)
            return it.key();
    }
    return -1;
}

QHash<int, QByteArray> SortFilterProxyModel::roleNames() const
{
    if (QAbstractItemModel *source = sourceModel())
        return source->roleNames();
    return QHash<int, QByteArray>();
}

bool SortFilterProxyModel::filterAcceptsRow(int sourceRow, const QModelIndex &sourceParent) const
{
    QRegExp rx = filterRegExp();
    if (rx.isEmpty())
        return true;
    QAbstractItemModel *model = sourceModel();
    if (filterRole().isEmpty()) {
        QHash<int, QByteArray> roles = roleNames();
        QHashIterator<int, QByteArray> it(roles);
        while (it.hasNext()) {
            it.next();
            QModelIndex sourceIndex = model->index(sourceRow, 0, sourceParent);
            QString key = model->data(sourceIndex, it.key()).toString();
            if (key.contains(rx))
                return true;
        }
        return false;
    }
    QModelIndex sourceIndex = model->index(sourceRow, 0, sourceParent);
    if (!sourceIndex.isValid())
        return true;
    QString key = model->data(sourceIndex, roleKey(filterRole())).toString();
    return key.contains(rx);
}

```

Q_PROPERTY()宏就是属性的声明：

- type 是指属性的类型，可以是 C++ 标准类型、类名、结构体、枚举等，name 就是属性的名字。
- READ 标出该属性的读函数 getFunction，Qt 属性的读函数通常省略 get 三个字母。
- WRITE 标出该属性的写函数 setFunction，中括号表示可选，写函数不是必须的。
- RESET 标出该属性的重置函数 resetFunction，重置函数将属性设为某个默认值，中括号表示可选，重置函数不是必须的。
- NOTIFY 标出该属性变化时发出的通知信号 notifySignal，中括号表示可选，这个信号不是必须的。

一个信号可以和另一个信号连接

```c++
QObject::connect(&a,SIGNAL(valueChanged(QString)),&b,SIGNAL(valueChanged(QString)));
```