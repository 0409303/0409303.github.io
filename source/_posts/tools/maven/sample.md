
```
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.microsoft.sam.SAMJobRunner</mainClass>
            <manifestEntries>
                <Class-Path>..</Class-Path>
            </manifestEntries>
        </transformer>

        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
            <resource>META-INF/cxf/bus-extensions.txt</resource>
        </transformer>

    </transformers>
</configuration>
```
