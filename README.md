# soLid-example

```php

<?php

class ScheduleDto
{
    /** @var string */
    private $stationCode;
    /** @var string */
    private $startTime;
    /** @var string */
    private $endTime;

    public function __construct(string $stationCode, string $startTime, string $endTime)
    {
        $this->stationCode = $stationCode;
        $this->startTime = $startTime;
        $this->endTime = $endTime;
    }

    public function getStationCode(): string
    {
        return $this->stationCode;
    }

    public function getStartTime(): string
    {
        return $this->startTime;
    }

    public function getEndTime(): string
    {
        return $this->endTime;
    }

    public function toArray(): array
    {
        return [
            'stationCode' => $this->stationCode,
            'startTime' => $this->startTime,
            'endTime' => $this->endTime,
        ];
    }
}

interface DataProviderContract
{
    /**
     * @return ScheduleDto[]
     */
    public function getScheduleItems(): array;
}


class GIS2DataProvider implements DataProviderContract
{
    public function getScheduleItems(): array
    {
        return [
            new ScheduleDto(
                'BWM Motors',
                '08:00',
                '17:00'
            ),

            new ScheduleDto(
                'Avto73',
                '09:00',
                '18:00'
            )
        ];
    }
}

class ScheduleRepository
{
    public function save(ScheduleDto $item): void
    {
        // save logic
        echo $this->saveLog($item) . PHP_EOL;
    }

    protected function saveLog(ScheduleDto $item): string
    {
        return 'saved ' . $item->getStationCode();
    }
}

class ScheduleParserService
{
    /** @var DataProviderContract */
    private $dataProvider;
    /** @var ScheduleRepository */
    private $scheduleRepository;

    public function __construct(DataProviderContract $dataProvider, ScheduleRepository $scheduleRepository)
    {
        $this->dataProvider = $dataProvider;
        $this->scheduleRepository = $scheduleRepository;
    }

    public function parse(): void
    {
        $items = $this->dataProvider->getScheduleItems();
        foreach ($items as $item) {
            $this->scheduleRepository->save($item);
        }
    }
}

class GoogleDataProvider implements DataProviderContract
{
    public function getScheduleItems(): array
    {
        return [
            new ScheduleDto(
                'Googled BWM Motors',
                '08:00',
                '17:00'
            ),

            new ScheduleDto(
                'Googled Avto73',
                '09:00',
                '18:00'
            )
        ];
    }
}

class AdvancedScheduleRepository extends ScheduleRepository
{
    protected function saveLog(ScheduleDto $item): string
    {
        return json_encode($item->toArray());
    }
}

$di = [
//    'scheduleRepository' => ScheduleRepository::class,
    'scheduleRepository' => AdvancedScheduleRepository::class,
    'dataProvider' => GIS2DataProvider::class,
//    'dataProvider' => GoogleDataProvider::class,
];

$parser = new ScheduleParserService(new $di['dataProvider'], new $di['scheduleRepository']);
$parser->parse();



```
